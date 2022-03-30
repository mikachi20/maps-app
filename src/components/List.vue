<template>
  <div class="lodge_list">
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
</template>

<script>
import { onAuthStateChanged } from "firebase/auth"
import { auth, db } from "@/firebase"
import { doc, setDoc, collection } from "firebase/firestore"
export default {
  data() {
    return {
      isFavorited: true,
    }
  },
  props: {
    list: {
      type: Object,
    },
    index: {
      type: Number,
    },
  },
  methods: {
    saveFavorite() {
      onAuthStateChanged(auth, (user) => {
        //メール認証をしているかどうか
        if (user) {
          const uid = user.uid
          const ref = doc(collection(db, "users", uid, "favorite"))
          setDoc(ref, {
            id: ref.id,
            ...this.lodgeLists,
          })
          console.log("お気に入りに追加しました。")
          this.isFavorited = false
        } else {
          console.log("お気に入りを追加出来ません。")
        }
      })
    },
  },
}
</script>

<style>
.lodge_list {
  display: flex;
  margin: 5px 50px;
  border-width: 1px;
  border-color: rgb(150, 150, 150);
  border-style: solid;
}
.lodge_contents {
  display: flex;
}
.lodge_photo {
  width: 192px;
  height: 135px;
  margin: 10px 20px 10px 10px;
}
.lodge_name {
  text-align: left;
  font-size: 20px;
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
  height: 30px;
  width: auto;
  margin-top: 15px;
  margin-left: auto;
  margin-right: 15px;
}
</style>
