<script>
	//import About from './about.svelte';
	import { initializeApp, getApps, getApp } from 'firebase/app';
	import { getFirestore, collection, onSnapshot } from 'firebase/firestore';
	import { firebaseConfig } from '$lib/firebaseConfig';
	import { browser } from '$app/env';

	const firebaseApp =
		browser && (getApps().length === 0 ? initializeApp(firebaseConfig) : getApp());

	const db = browser && getFirestore();
	let todos = [];

	const colRef = browser && collection(db, 'todos');
	const unsubscribe =
		browser &&
		onSnapshot(colRef, (querySnapshot) => {
			let fbTodos = [];
			querySnapshot.forEach((doc) => {
				let todo = { ...doc.data(), id: doc.id };
				fbTodos = [todo, ...fbTodos];
			});
			todos = fbTodos;
		});

	let task = '';

	const addTodo = () => {
		let todo = {
			task: task,
			isComplete: false,
			createdAt: new Date().toDateString
		};

		// @ts-ignore
		todos = [...todos, todo];
		task = '';
	};

	//$: console.table(todos);

	const markTodoAsComplete = (index) => {
		todos[index].isComplete = !todos[index].isComplete;
	};

	const deleteTodo = (index) => {
		let deleteItem = todos[index];
		todos = todos.filter((item) => item != deleteItem);
	};
</script>

<input type="text" placeholder="Add a Task" bind:value={task} />
<button on:click={addTodo}>Add</button>

<ol>
	{#each todos as todo, index}
		<li class:complete={todo.isComplete}>
			<span>
				{todo.task}
			</span>
			<span><button on:click={() => markTodoAsComplete(index)}>✓</button> </span>
			<span><button on:click={() => deleteTodo(index)}>✘</button> </span>
		</li>
	{:else}
		<p>NO Todos Found !</p>
	{/each}
</ol>

<style>
	.complete {
		text-decoration: line-through;
	}
</style>
