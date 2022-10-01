<script>
    import { initializeApp } from "firebase/app";
    import { getFirestore, collection, onSnapshot, doc, addDoc, updateDoc, deleteDoc } from "firebase/firestore";
    import { firebaseConfig } from "$lib/firebaseConfig";

    // initialize Firebase with config document we created
    const firebaseApp = initializeApp(firebaseConfig);
    // Initialize Cloud Firestore and get a reference to the service
    const db = getFirestore(firebaseApp);

    const colRef = collection(db, "todos");

    // start off with empty todo list
    let todos = [];

    const unsubscribe = onSnapshot(colRef, (querySnapshot) => {
        let fbTodos = [];
        querySnapshot.forEach((doc) => {
            let todo = {...doc.data(), id: doc.id}
            fbTodos = [todo, ...fbTodos];
        });
        todos = fbTodos;
    })

    // console.log(firebaseApp, db);
    
    
    let task = "";

    const addTodo = async () => {
        // let todo = {
        //     task: task,
        //     isComplete: false,
        //     createdAt: new Date(),
        // };
        // check if input is non empty
        if (task !== "") {
            // todos = [...todos, todo]
            const docRef = await addDoc(collection(db, "todos"), {
                task: task,
                isComplete: false,
                createdAt: new Date(),
            });
        } 
        
        // clear previous input
        task = "";
    };

    // mark todo as complete when clicked
    const markTodoAsComplete = async (item) => {
        //todos[index].isComplete = !todos[index].isComplete;
        // update item.isComplete in firebase
        await updateDoc(doc(db, "todos", item.id), {
            isComplete: !item.isComplete
        });
    };

    const deleteTodo = async (id) => {
        // delete by item id
        await deleteDoc(doc(db, "todos", id));
    }

    // add todo if user pressed enter key
    const keyIsPressed = (event) => {
        if (event.key === "Enter") {
            addTodo();
        }
    }

    // $: console.table(todos);
</script>

<input type="text" placeholder="Add a task" bind:value={task}>
<button on:click={addTodo}>Add</button>

<ol>
    {#each todos as item}
        <li class:complete={item.isComplete}>
        <span>
            {item.task}
        </span>
        <span>
            <button on:click={() => markTodoAsComplete(item)}>✓</button>
            <button on:click={() => deleteTodo(item.id)}>✗</button>
        </span>
        </li>
        {:else}
        <p>No todos found</p>
    {/each}
</ol>


<svelte:window on:keydown={keyIsPressed} />

<style>
    .complete {
        text-decoration: line-through;
    }
</style>