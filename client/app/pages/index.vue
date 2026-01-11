<template>
<div id="app">
<h1>連絡先アプリ</h1>
<div class="new">
<h2>新規作成</h2>
<div class="name">
<label for="name">お名前：</label>
<input type="text" name="name" id="name" v-model="newName" />
</div>
<div class="email">
<label for="email">メールアドレス：</label>
<input type="email" name="email" id="email" v-model="newEmail" />
</div>
<button @click="insertContact">新規作成</button>
</div>
<div class="table">
<h2>連絡先リスト</h2>
<table>
<tr>
<th>ID</th>
<th>NAME</th>
<th>EMAIL</th>
<th>UPDATE</th>
<th>DELETE</th>
</tr>
<tr v-for="item in contactLists" :key="item.id">
<td>{{ item.id }}</td>
<td><input type="text" v-model="item.name" /></td>
<td><input type="email" v-model="item.email" /></td>
<td>
<button @click="updateContact(item.id, item.name, item.email)">
更新
</button>
</td>
<td>
<button @click="deleteContact(item.id)">削除</button>
</td>
</tr>
</table>
</div>
</div>
</template>


<script setup>
// runtimeConfigからURLを取得
const config = useRuntimeConfig();
const apiURL = config.public.apiBase;

// データの状態を管理（refを使用）
const newName = ref("");
const newEmail = ref("");
const contactLists = ref([]);

// 1. データ取得
const getContact = async () => {
    try {
    // $axios.get ではなく $fetch を使用
    const res = await $fetch(apiURL);
    contactLists.value = res.data; // Laravel側が {data: [...]} で返す想定
    } catch (e) {
    console.error("取得エラー", e);
    }
};

// 2. 新規作成
const insertContact = async () => {
    try {
    await $fetch(apiURL, {
        method: 'POST',
        body: { name: newName.value, email: newEmail.value }
    });
    newName.value = "";
    newEmail.value = "";
    getContact(); // 再取得
    } catch (e) {
    alert("作成に失敗しました");
    }
};

// 3. 更新
const updateContact = async (id, name, email) => {
    try {
    await $fetch(`${apiURL}${id}`, {
        method: 'PUT',
        body: { name, email }
    });
    alert("更新しました");
    getContact();
    } catch (e) {
    alert("更新エラー");
    }
};

// 4. 削除
const deleteContact = async (id) => {
    if(!confirm("削除しますか？")) return;
    try {
    await $fetch(`${apiURL}${id}`, {
        method: 'DELETE'
    });
    getContact();
    } catch (e) {
    alert("削除エラー");
    }
};

// 画面が表示された時に実行
onMounted(() => {
    getContact();
});
</script>

<style>
table,
td,
th {
border: 1px solid #000;
border-collapse: collapse;
text-align: center;
}
td,
th {
padding: 5px;
}
th {
background: #f0e6cc;
}
</style>