<script setup>
import { ref } from "vue";
import { statuses } from "@/const/statuses";

let items = ref(JSON.parse(localStorage.getItem("items")) || []);
//TodoInput.vueと同じ
//ローカルストレージから受け取るデータはid,content,limit,state,onEditなど
//refを指定しないと変数が変わっても画面に反映されない
let inputContent = ref("");
let inputLimit = ref("");
let inputState = ref("");
//編集モードの入力欄の値を扱う変数をrefで宣言

const onEdit = (id) => {
  //idを受け取ったら実行される
  inputContent.value = items.value[id].content;
  inputLimit.value = items.value[id].limit;
  inputState.value = items.value[id].state;
  items.value[id].onEdit = true;
  // =true にしていたが、編集モードの時に編集ボタンを押しても変数の値がtrueのままなことに気づき !items.value[id].onEdit にした
  //onUpdateでfalseにするので =true に戻した
};

let isErrMsg = ref(false);

const onUpdate = (id) => {
  if (inputContent.value == "" || inputLimit.value == "") {
    isErrMsg.value = true;
    return;
  }

  const newItem = {
    id: id, //受け取ったID
    content: inputContent.value,
    limit: inputLimit.value,
    state: inputState.value,
    onEdit: false, //編集が完了するのでfalseにしておく
  };
  items.value.splice(id, 1, newItem);
  //items配列のうち更新したい要素を上記のnewItemに置き換える
  //今itemsにあるデータをidで特定してその一つを新しく編集で入力したデータ(newItem)に置き換える

  localStorage.setItem("items", JSON.stringify(items));
  isErrMsg.value = false; //自動リロードがないため手動でfalseを設定？
};
</script>

<template>
  <div>
    <p v-if="isErrMsg">タスク・期限を両方入力してください</p>
    <table>
      <!-- タスクを画面に表示させるためのtable -->
      <!-- ローカルストレージから受け取ったデータをtableで表示 -->
      <tr>
        <th class="th-id">ID</th>
        <th class="th-value">やること</th>
        <th class="th-limit">期限</th>
        <th class="th-state">状態</th>
        <th class="th-edit">編集</th>
        <th class="th-delete">削除</th>
      </tr>
      <!-- タスクの件数分表示 -->
      <tr v-for="item in items" :key="item.id">
        <!-- 変数名inリスト名 -->
        <td>
          {{ item.id }}
        </td>
        <td>
          <span v-if="!item.onEdit"> {{ item.content }}</span>
          <input v-else v-model="inputContent" type="text" />
        </td>
        <td>
          <span v-if="!item.onEdit"> {{ item.limit }}</span>
          <input v-else v-model="inputLimit" type="date" />
        </td>
        <td>
          <span v-if="!item.onEdit"> {{ item.state.value }}</span>
          <select v-else v-model="inputState">
            <option
              v-for="state in statuses"
              :key="state.id"
              :value="state"
              :selected="state.id == item.state.id"
            >
              {{ state.value }}
            </option>
            <!-- :selectedは初期値のことで、現在のステータス(とIDが一致するもの)を表示 -->
          </select>
        </td>
        <td>
          <button v-if="!item.onEdit" @click="onEdit(item.id)">編集</button>
          <!-- どのタスクを編集モードにするか判別させるために引数でidを持たせる -->
          <button v-else @click="onUpdate(item.id)">完了</button>
        </td>
        <td><button>削除</button></td>
      </tr>
    </table>
  </div>
</template>

<style scoped></style>
