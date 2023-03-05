<script>
  import { firebaseConfig } from "./lib/firebaseconfig";
  import { initializeApp } from "firebase/app";
  import {
    doc,
    addDoc,
    collection,
    getDoc,
    getFirestore,
    onSnapshot,
    setDoc,
    deleteDoc,
  } from "firebase/firestore";

  const app = initializeApp(firebaseConfig);
  const firestore = getFirestore(app);

  let tasks = [];

  let curr = "";

  //   let firetodos = get();
  onSnapshot(collection(firestore, "todos/username/privatetodos"), (col) => {
    tasks = [];
    col.forEach((doc) => {
      const new_task = {
        id: doc.id,
        name: doc.data().name,
        iscompleted: doc.data().iscompleted,
        created_at: doc.data().created_at,
      };
      tasks = [...tasks, new_task];
    });
  });

  const AddTask = async () => {
    if (curr == "") return;
    const new_task = {
      name: curr,
      iscompleted: false,
      created_at: new Date(),
    };
    await addDoc(
      collection(firestore, "todos/username/privatetodos"),
      new_task
    );
    curr = "";
  };

  const ToggleTask = async (item) => {
    await setDoc(
      doc(firestore, `todos/username/privatetodos/${item.id}`),
      { iscompleted: !item.iscompleted },
      { merge: true }
    );
  };
  const DeleteTask = async (item) => {
    await deleteDoc(doc(firestore, `todos/username/privatetodos/${item.id}`));
  };
</script>

<h1>Tasks</h1>

<input type="text" placeholder="Enter your task" bind:value={curr} />
<button on:click={AddTask}>Add</button>

<ol>
  {#each tasks as task}
    <li>
      <span class:complete={task.iscompleted}>
        {task.name}
      </span>
      <span>
        <button
          on:click={() => {
            ToggleTask(task);
          }}>✓</button
        >
        <button
          on:click={() => {
            DeleteTask(task);
          }}>✕</button
        >
      </span>
    </li>
  {:else}
    <p>No tasks found</p>
  {/each}
</ol>

<style>
  .complete {
    text-decoration: line-through;
  }
</style>
