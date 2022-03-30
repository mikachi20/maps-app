<template>
  <head>
    <title>Add Map</title>
  </head>
  <body>
    <h3>My Google Maps</h3>
    <!--マップを表示するところ-->
    <div id="map" ref="map" />

    <!-- ここからMyPage.vueから引用 -->
    <h3>お気に入り</h3>
    <div class="lodge_list" v-for="(favList, index) in favLists" :key="index">
      <img class="lodge_photo" :src="favList.photo" />
      <div>
        <div class="lodge_name">{{ favList.name }}</div>
        <div class="lodge_kana">{{ favList.kana }}</div>
        <div class="lodge_address">{{ favList.address }}</div>
        <div class="lodge_phone">TEL : {{ favList.phone }}</div>
        <div class="lodge_url">詳細 : {{ favList.url }}</div>
        <div class="lodge_latlng"></div>
      </div>

      <button class="favorite-icon" @click="deleteFavorite(index)">
        {{ favList.buttonText }}
      </button>
    </div>
    <div v-if="isLogin"></div>
    <div v-else>
      ログインまたはアカウントの作成をするとお気に入り登録ができます。
    </div>
  </body>
</template>

<script>
import { onAuthStateChanged } from "firebase/auth"
import { auth, db } from "@/firebase"
import { collection, getDocs, doc, deleteDoc } from "firebase/firestore"
export default {
  data: function () {
    return {
      latLngs: [{ lat: 35.6621, lng: 139.70378, name: "渋谷" }],
      favLists: [],
      uid: null,
      isLogin: false,
    }
  },
  methods: {
    makeArray: function (latLng, map) {
      const newMarker = new window.google.maps.Marker({
        position: latLng,
        map: map,
      })

      let Lat = latLng.lat()
      let Lng = latLng.lng()
      let newLatlng = { lat: Lat, lng: Lng }
      this.latLngs.push(newLatlng)

      newMarker.setMap(map)
      newMarker.addListener("click", () => {
        //this.deleteButton(p.latLng, marker)
        newMarker.setMap(null)
        if (this.latLngs.includes(newLatlng)) {
          this.latLngs.splice(newLatlng, 1)
        }
      })
    },

    // ここからMyPageから引用
    deleteFavorite: async function (index) {
      const res = await fetch(
        "https://app.rakuten.co.jp/services/api/Travel/SimpleHotelSearch/20170426?format=json&datumType=1&latitude=35.6065914&longitude=139.7513225&applicationId=1064072543465509183"
      )
      const value = await res.json()
      onAuthStateChanged(auth, (user) => {
        //メール認証をしているかどうか
        if (user) {
          const uid = user.uid
          const favid = value.hotels[index].hotel[0].hotelBasicInfo.telephoneNo
          const ref = doc(collection(db, "users", uid, "favorite"), favid)
          deleteDoc(ref)
          console.log("お気に入りを取り消しました。")
        } else {
          console.log("お気に入りを取り消し出来ません。")
        }
      })
    },
  },
  created() {
    onAuthStateChanged(auth, (user) => {
      if (user) {
        this.isLogin = true
        const uid = user.uid
        getDocs(collection(db, "users", uid, "favorite")).then((snapshot) => {
          snapshot.forEach((doc) => {
            this.favLists.push({
              ...doc.data(),
              buttonText: "削除",
            })
          })
        })
      } else {
        this.isLogin = false
      }
    })
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
    /*mountedの中一部抜粋*/
    //初期のマップ配置
    let timeCount = setInterval(() => {
      if (window.mapCompleted === true) {
        clearInterval(timeCount)
        const map = new window.google.maps.Map(this.$refs.map, {
          center: this.latLngs[0],
          zoom: 12,
          mapType: "default",
        })
        map.addListener("click", (e) => {
          this.makeArray(e.latLng, map)
        })

        //最初の配列をすべて表示するためのfor文
        //firebaseとの連結に
        for (let i = 0; i < this.latLngs.length; i++) {
          const marker = new window.google.maps.Marker({
            position: this.latLngs[i],
            map: map,
          })
          marker.setMap(map)
          marker.addListener("click", () => {
            marker.setMap(null)
            const firstlatLng = this.latLngs[i]
            if (this.latLngs.includes(firstlatLng)) {
              this.latLngs.splice(firstlatLng, 1)
            }
          })
        }
      }
    }, 500)
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
