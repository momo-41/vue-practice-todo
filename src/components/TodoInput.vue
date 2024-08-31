<script setup>
import { ref } from "vue";
import { statuses } from "@/const/statuses";

const input = ref("");
const inputDate = ref("");

const isErrMsg = ref(false);
//エラーメッセージを出すか出さないかの判別をするための変数

const onSubmitForm = () => {
  if (input.value == "" || inputDate.value == "") {
    isErrMsg.value = true;
    event.preventDefault();
    //エラーメッセージを表示するときはリロードをしないようにする
    //formを送信するとリロードされる仕様になっている
    return;
  }

  const items = JSON.parse(localStorage.getItem("items")) || [];
  //ローカルストレージからのデータの取得はgetItem(key)
  //ローカルストレージには文字列(JSON)形式で保存されているのでJSON.parse()で配列で扱えるように変換
  //ローカルストレージにデータがなかった場合にはからの配列が返されるように [] を指定しておく(この後pushするために必要)

  const newItem = {
    id: items.length, //初めはitemsは無いので0から始まる連番になる
    content: input.value,
    limit: inputDate.value,
    state: statuses.NOT_START,
    onEdit: false, //編集中フラグ
  }; //ローカルストレージに保存するタスクの情報

  items.push(newItem); //newItemをitemsに追加

  localStorage.setItem("items", JSON.stringify(items));
  //ローカルストレージへの保存はsetItem(key,value)
  //配列のままではローカルストレージに保存できないのでJSON.stringify()でJSON形式に変換
};
</script>

<template>
  <div>
    <p v-if="isErrMsg">タスク・期限を両方入力してください</p>
    <!-- isErrMsgがtrueの時だけ上記の文字が表示される -->
    <form @submit="onSubmitForm">
      <label>やること<input type="text" v-model="input" /></label>
      <label>期限<input type="date" v-model="inputDate" /></label>
      <input type="submit" value="登録！" />
    </form>
  </div>
</template>

<style scoped></style>
