<script setup>
const items = JSON.parse(localStorage.getItem("items")) || [];
//TodoInput.vueと同じ
//ローカルストレージから受け取るデータはid,content,limit,state,onEditなど
</script>

<template>
  <div>
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
        <td>{{ item.id }}</td>
        <td>{{ item.content }}</td>
        <td>{{ item.limit }}</td>
        <td>{{ item.state.value }}</td>
        <td>{{ item.id }}</td>
        <td><button>編集</button></td>
        <td><button>削除</button></td>
      </tr>
    </table>
  </div>
</template>

<style scoped></style>
