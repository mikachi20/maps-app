<template>
  <head>
    <title>Add Map</title>
  </head>
  <body>
    <div>
      <p>ログイン状態: {{ authState }}</p>
      <p>メール認証: {{ emailVerified }}</p>
      <button @click="signOut">サインアウト</button>
    </div>
    <h3>Google Map</h3>
    <p>マップをタッチして周辺の宿を探そう！</p>
    <!--マップを表示するところ-->
    <div id="map" ref="map" />
    <div></div>

    <!-- ここからAPIから引用 -->
    <section>
      <div>
        <input
          type="text"
          class="num"
          v-bind:value="latitude"
          v-on:input="changeLatitude"
        />緯度：{{ latitude }}
      </div>
      <div>
        <input
          type="text"
          v-bind:value="longitude"
          v-on:input="changeLongitude"
        />経度：{{ longitude }}
      </div>
    </section>

    <section>
      <button v-on:click="getLongitude">---宿の検索---</button>
    </section>

    <!-- ここからFavorite -->

    <h3>宿泊施設</h3>
    <div>
      <div
        class="lodge_list"
        v-for="(list, index) in hotelList"
        v-bind:key="index"
      >
        <img class="lodge_photo" :src="list.photo" />
        <div>
          <div class="lodge_name">{{ list.name }}</div>
          <div class="lodge_kana">{{ list.kana }}</div>
          <div class="lodge_address">{{ list.address }}</div>
          <div class="lodge_phone">TEL : {{ list.phone }}</div>
          <div class="lodge_url">詳細 : {{ list.url }}</div>
        </div>
        <button class="favorite-icon" @click="saveFavorite(index)">
          {{ list.buttonText }}
        </button>
      </div>
    </div>
  </body>
</template>

<script>
//Favoriteから引用
import { signOut, onAuthStateChanged } from "firebase/auth"
import { auth, db } from "@/firebase"
import { doc, setDoc, collection, deleteDoc } from "firebase/firestore"

export default {
  data: function () {
    return {
      latLngs: [{ lat: 35.6621, lng: 139.70378, name: "渋谷" }],
      //memoWindow: false,
      //緯度経度の初期値設定
      latitude: 35.6065914,
      longitude: 139.7513225,
      authState: "",
      emailVerified: "",
      uid: null,
      hotelList: [],
    }
  },
  methods: {
    //pinをつける部分
    pinSet: function (latLng) {
      let Lat = latLng.lat()
      let Lng = latLng.lng()
      this.latLngs.splice(0, 1, { lat: Lat, lng: Lng, name: "" })

      let newMap = new window.google.maps.Map(this.$refs.map, {
        center: this.latLngs[0],
        zoom: 12,
        mapType: "default",
      })
      /*const newMarker = */ new window.google.maps.Marker({
        position: this.latLngs[0],
        map: newMap,
      })
      this.latitude = Lat
      this.longitude = Lng

      //二回目以降が付かなかったので、こちらで
      newMap.addListener("click", (e) => {
        this.pinSet(e.latLng)
        this.latitude = e.latLng.lat()
        this.longitude = e.latLng.lng()
      })

      /* //情報用クリック
      newMarker.addListener("click", (p) => {
        this.info(p.latLng, newMarker, newMap)
      })*/
    },

    //APIから引用
    changeLatitude(event) {
      this.latitude = event.target.value
    },
    changeLongitude(event) {
      this.longitude = event.target.value
    },
    getLongitude: function () {
      // 3------------------------------------------------------------------------------latitudeとlongitudeのnumberg型の変換が必要
      console.log(this.latitude)
      this.latitude = parseFloat(this.latitude)
      this.longitude = parseFloat(this.longitude)
      // console.log(typeof num1)
      // console.log(num2)
      // console.log(typeof num2)
      console.log(typeof this.latitude)
      console.log(typeof this.longitude)
      //ボタンクリック後に緯度経度データを入手する
      fetch(
        "https://app.rakuten.co.jp/services/api/Travel/SimpleHotelSearch/20170426?format=json&datumType=1&latitude=" +
          this.latitude +
          "&longitude=" +
          this.longitude +
          "&applicationId=1064072543465509183"
      ) //1----------------------------------------------------------------------------緯度経度を変えて毎回動的に表示するには
        //2------------------------------------------------------------------------------緯度経度の指定方法に決まりがある？世界測地系、単位は度で指定すること。世界地図系とかなにかルールあり？
        .then((response) => {
          return response.json()
        })
        .then((value) => {
          console.log(value)
          console.log(value.hotels[0].hotel[0].hotelBasicInfo)
          this.infomations = value
          for (let i = 0; i < value.hotels.length; i++) {
            const hotelName = value.hotels[i].hotel[0].hotelBasicInfo.hotelName
            const hotelKana =
              value.hotels[i].hotel[0].hotelBasicInfo.hotelKanaName
            const hotelUrl =
              value.hotels[i].hotel[0].hotelBasicInfo.hotelInformationUrl
            const hotelAddres1 =
              value.hotels[i].hotel[0].hotelBasicInfo.address1
            const hotelAddres2 =
              value.hotels[i].hotel[0].hotelBasicInfo.address2
            const hotelPhone =
              value.hotels[i].hotel[0].hotelBasicInfo.telephoneNo
            const hotelPhoto =
              value.hotels[i].hotel[0].hotelBasicInfo.hotelImageUrl
            const hotelLat = value.hotels[i].hotel[0].hotelBasicInfo.latitude
            const hotelLng = value.hotels[i].hotel[0].hotelBasicInfo.longitude
            const buttonText = "追加"
            if (
              !(hotelName, hotelUrl, hotelAddres1, hotelAddres2, hotelPhone)
                .length
            ) {
              return
            } else {
              this.hotelList.push({
                name: hotelName,
                kana: hotelKana,
                url: hotelUrl,
                address: hotelAddres1 + hotelAddres2,
                phone: hotelPhone,
                photo: hotelPhoto,
                buttonText: buttonText,
                hotelLating: hotelLat,
                hotelLng: hotelLng,
              })
            }
          }
        })
        .catch((error) => {
          console.error(error)
          console.log(error)
          this.infomations = error
        })
    },

    //Favoriteから引用
    signOut() {
      signOut(auth)
        //ログアウト成功した場合はログイン画面へ
        .then(() => {
          this.$router.push("/login")
        })
        //失敗した場合はエラーメッセージを画面に出力
        .catch((e) => (this.error = e.message))
    },
    isFavorite(index) {
      // ブックマークリスト内にbook idがあればtrue それ以外はfalse
      return this.hotelList[index]
    },
    saveFavorite: async function (index) {
      if (this.hotelList[index].buttonText === "追加") {
        const res = await fetch(
          "https://app.rakuten.co.jp/services/api/Travel/SimpleHotelSearch/20170426?format=json&datumType=1&latitude=35.6065914&longitude=139.7513225&applicationId=1064072543465509183"
        )
        const value = await res.json()
        const hotelName = value.hotels[index].hotel[0].hotelBasicInfo.hotelName
        const hotelKana =
          value.hotels[index].hotel[0].hotelBasicInfo.hotelKanaName
        const hotelUrl =
          value.hotels[index].hotel[0].hotelBasicInfo.hotelInformationUrl
        const hotelAddres1 =
          value.hotels[index].hotel[0].hotelBasicInfo.address1
        const hotelAddres2 =
          value.hotels[index].hotel[0].hotelBasicInfo.address2
        const hotelPhone =
          value.hotels[index].hotel[0].hotelBasicInfo.telephoneNo
        const hotelPhoto =
          value.hotels[index].hotel[0].hotelBasicInfo.hotelImageUrl
        const hotelLat = value.hotels[index].hotel[0].hotelBasicInfo.latitude
        const hotelLng = value.hotels[index].hotel[0].hotelBasicInfo.longitude

        onAuthStateChanged(auth, (user) => {
          //メール認証をしているかどうか
          if (user) {
            const uid = user.uid
            const ref = doc(
              collection(db, "users", uid, "favorite"),
              hotelPhone
            )
            setDoc(ref, {
              name: hotelName,
              kana: hotelKana,
              url: hotelUrl,
              address: hotelAddres1 + hotelAddres2,
              phone: hotelPhone,
              photo: hotelPhoto,
              id: ref.id,
              hotelLating: hotelLat,
              hotelLng: hotelLng,
            })
            console.log("お気に入りに追加しました。")
            this.hotelList[index].buttonText = "済み"
          } else {
            console.log("お気に入りを追加出来ません。")
          }
        })
      } else {
        const res = await fetch(
          "https://app.rakuten.co.jp/services/api/Travel/SimpleHotelSearch/20170426?format=json&datumType=1&latitude=35.6065914&longitude=139.7513225&applicationId=1064072543465509183"
        )
        const value = await res.json()
        onAuthStateChanged(auth, (user) => {
          //メール認証をしているかどうか
          if (user) {
            const uid = user.uid
            const favid =
              value.hotels[index].hotel[0].hotelBasicInfo.telephoneNo
            const ref = doc(collection(db, "users", uid, "favorite"), favid)
            deleteDoc(ref)
            console.log("お気に入りを取り消しました。")
            this.hotelList[index].buttonText = "追加"
          } else {
            console.log("お気に入りを取り消し出来ません。")
          }
        })
      }
    },
  },

  mounted: function () {
    if (!window.mapCompleted) {
      window.mapCompleted = true
      const script = document.createElement("script")
      script.src =
        //API入力場所
        script.async = true
      document.head.appendChild(script)
    }

    window.initMap = () => {
      window.mapCompleted = true
    }
    //初期のマップ配置
    let timeCount = setInterval(() => {
      if (window.mapCompleted === true) {
        clearInterval(timeCount)
        const map = new window.google.maps.Map(this.$refs.map, {
          center: this.latLngs[0],
          zoom: 8,
          mapType: "default",
        })
        //最初の配列をすべて表示するためのfor文
        //firebaseとの連結に
        for (let i = 0; i < this.latLngs.length; i++) {
          /*const marker = */ new window.google.maps.Marker({
            position: this.latLngs[i],
            map: map,
          })
        }
        //マップクリック時にpinSet,info内の処理を行う
        map.addListener("click", (e) => {
          this.pinSet(e.latLng)
        })
      }
    }, 500)
  },
  created: function () {
    //現在ログインしているユーザの情報を取得
    onAuthStateChanged(auth, (user) => {
      //メール認証をしているかどうか
      if (user) {
        this.authState = "ログイン"
        this.emailVerified = user.emailVerified ? "済" : "未"
      } else {
        this.authState = "ログアウト"
        this.emailVerified = "-"
      }
    })
  },
}
</script>

<style>
#map {
  height: 600px;
  width: 100%;
}
.lodge_list {
  display: flex;
  margin: 5px 50px;
  border-width: 1px;
  border-color: rgb(150, 150, 150);
  border-style: solid;
  position: relative;
}
.lodge_contents {
  display: flex;
}
.lodge_photo {
  width: 160px;
  height: 120px;
  margin: 10px 10px 10px 10px;
}
.lodge_name {
  text-align: left;
  font-size: 17px;
  font-weight: bolder;
  margin-top: 10px;
  margin-left: 10px;
}
.lodge_kana {
  text-align: left;
  font-size: 10px;
  margin-left: 10px;
}
.lodge_address {
  text-align: left;
  font-size: 15px;
  margin-left: 10px;
}
.lodge_phone {
  text-align: left;
  font-size: 15px;
  margin-left: 10px;
}
.lodge_url {
  text-align: left;
  font-size: 15px;
  margin-left: 10px;
  margin-bottom: 10px;
}
.favorite-icon {
  position: absolute;
  right: 10px;
  top: 10px;
}
</style>
