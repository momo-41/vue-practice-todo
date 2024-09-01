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
  //上記で編集モードにしてから以下で isOnEditOther=false にして
  //今編集モードになっているもの以外を対象として、それらのonEditがtrueになっているか以下で確かめる

  let isOnEditOther = false; //他にonEditがtrueのものがないか確認する変数
  items.value.map((item) => {
    if (item.onEdit) {
      isOnEditOther = true;
      return;
    }
  });
  if (isOnEditOther) {
    errMsg.value = "他に編集中のタスクがあります";
    isErrMsg.value = true;
    return;
  }
};

let errMsg = ref(""); //エラーメッセージが入る変数
let isErrMsg = ref(false);
const onUpdate = (id) => {
  if (inputContent.value == "" || inputLimit.value == "") {
    errMsg.value = "タスクの内容と期限を入力してください";
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

let isShowModal = ref(false);
let deleteItemId = ""; //削除対象のItemのID
let deleteItemContent = ref();
const showDeleteModal = (id) => {
  isShowModal.value = true;
  deleteItemId = id;
  deleteItemContent = items.value[id].content;
};

const onDeleteItem = () => {
  items.value.splice(deleteItemId, 1); //対象タスクの削除。deleteItemIdの番号のタスクを取り除く処理

  items.value = items.value.map((item, index) => ({
    id: index,
    content: item.content,
    limit: item.limit,
    state: item.state,
    onEdit: item.onEdit,
  }));

  localStorage.setItem("items", JSON.stringify(items.value));
  isShowModal.value = false;
};

const onHideModal = () => {
  isShowModal.value = false;
};

const today = new Date();

const sortByLimit = () => {
  items.value.sort((a, b) => new Date(a.limit) - new Date(b.limit)); //日付の比較
  localStorage.setItem("items", JSON.stringify(items.value));
};

const sortById = () => {
  items.value.sort((a, b) => a.id - b.id); //日付の比較
  localStorage.setItem("items", JSON.stringify(items.value));
};
</script>

<template>
  <div>
    <p v-if="isErrMsg">{{ errMsg }}</p>
    <div class="modal" v-if="isShowModal">
      <div class="modal-content">
        <p>{{ deleteItemContent }}を削除してもよろしいでしょうか</p>
        <button @click="onDeleteItem()">はい</button>
        <button @click="onHideModal()">キャンセル</button>
      </div>
    </div>
    <table>
      <!-- タスクを画面に表示させるためのtable -->
      <!-- ローカルストレージから受け取ったデータをtableで表示 -->
      <tr>
        <th class="th-id">ID<button @click="sortById()">↓</button></th>
        <th class="th-value">やること</th>
        <th class="th-limit">期限 <button @click="sortByLimit()">↓</button></th>
        <th class="th-state">状態</th>
        <th class="th-edit">編集</th>
        <th class="th-delete">削除</th>
      </tr>
      <!-- タスクの件数分表示 -->
      <tr
        v-for="item in items"
        :key="item.id"
        :class="{ red: new Date(item.limit) < today }"
      >
        <!-- trueだったらredになる -->
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
        <td>
          <button @click="showDeleteModal(item.id)">削除</button>
        </td>
      </tr>
    </table>
  </div>
</template>

<style scoped>
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background: #fff;
  padding: 20px;
  border-radius: 8px;
}

.red {
  color: red;
}
</style>
