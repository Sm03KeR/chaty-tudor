<template>
<div>
  <div class="view login" v-if="state.username === '' || state.username === null">
      <form class="login-form" @submit.prevent="Login">
        <div class="form-inner">
          <h1>Login to VueChat 👋</h1>
          <label for="username">Username</label>

          <input 
          type="text" 
          placeholder="Your username..." 
          v-model="inputUsername">

          <input type="submit" value="Login">
		  <p style="text-align:center; opacity:.5">Build by *Tudor Crisan*</p>

        </div>
      </form>
  </div>
  
  <div class="view chat" v-else>

    <header>
	  <h3>Welcome, {{state.username}} 👋</h3>
      <button title="Logout" class="logout" @click="Logout"><svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" aria-hidden="true" role="img" width="2em" height="2em" preserveAspectRatio="xMidYMid meet" viewBox="0 0 48 48"><g fill="none" stroke="#333" stroke-width="4" stroke-linecap="round" stroke-linejoin="round"><path d="M23.992 6H6v36h18M33 33l9-9l-9-9M16 23.992h26"/></g></svg></button>
    </header>

    <section class="chat-box">
      <div 
	  v-for="i in state.messages" 
	  :key="i.key" 
	  :class="(i.username == state.username ? 'message current-user' : 'message')" >
		<div class="message-inner">
			<div class="username">{{i.username}}</div>
			<div class="content">{{i.content}}</div>
		</div>
	  </div>
    </section>

    <footer>
      <form @submit.prevent="SendMessage">
        <input type="text" placeholder="Write a message..." v-model="inputMessage">
        <button type="submit"><svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" aria-hidden="true" role="img" width="1.3em" height="1.3em" preserveAspectRatio="xMidYMid meet" viewBox="0 0 512 512"><path fill="lightgray" d="M474.444 19.857a20.336 20.336 0 0 0-21.592-2.781L33.737 213.8v38.066l176.037 70.414L322.69 496h38.074l120.3-455.4a20.342 20.342 0 0 0-6.62-20.743zM337.257 459.693L240.2 310.37l149.353-163.582l-23.631-21.576L215.4 290.069L70.257 232.012L443.7 56.72z"/></svg></button>
      </form>
    </footer>
    
  </div>
</div>
</template>

<script>

import { reactive, onMounted, ref } from 'vue';
import db from './db';

export default {
  setup() {  //functie predefinita, cand se schimba ceva in ea, se actualizeaza site ul
    const inputUsername = ref(""); //cand tastez ceva in input ul de la username, se stocheaza aici
	const inputMessage = ref("");


    const state = reactive({ //reactive o modalitate de a salva niste date
      username: "",
      messages: []
    })
    //console.log(state);

    const Login = () => {
      if(inputUsername.value == "" || inputUsername.value == null) {
          alert('Please insert a username');
      } else {
          state.username = inputUsername.value;
          inputUsername.value = "";
      } 
    }

	const SendMessage = () => { // trimit datele la server, firebase
		const messagesRef = db.database().ref("messages"); //asta e baza de date

		if(inputMessage.value === "" || inputMessage.value === null) {
			alert('plase add a message');
		}

		const message = {
			username: state.username,
			content: inputMessage.value
		}

		messagesRef.push(message); //bagam mesajul in baza de date
		inputMessage.value = "";
		//window.scrollTo(0, document.body.scrollHeight || document.documentElement.scrollHeight);
	}

	onMounted(()=> {  //functie predefinita, de fiecare data cand se schimba ceva in app.vue, se apeleaza functia
		const messagesRef = db.database().ref("messages"); //baza de date firebase

		messagesRef.on('value', snapshot => { //daca se schimba ceva in baza de date
			const data = snapshot.val(); //data, is se atribuie toate mesajele salvate in firebase
			let messages = []; //in acest array iau mesajele stocate in firebase

			Object.keys(data).forEach(key => { //trev prin toate mesajele stocate in firebase, si dau push la un object cu id username si content in array ul messages
				messages.push({
					id: key,
					username: data[key].username,
					content: data[key].content
				});
			});

			state.messages = messages; //array ul cu mesajele il bag in state.messages, ca sa il punem in UI

			//console.log(data);
			console.log(Object.keys(data)[0]);
			//console.log(messages[0].id);

			if(Object.keys(data).length > 100) { //daca depaseste un nr limita de mesaje, se sterge primul mesaj, si tot asa
				db.database().ref("messages/" + Object.keys(data)[0]).remove();
			}
		});
	})
	

	const Logout = () => {
		state.username = "";
	}

    return{
      inputUsername,
      Login,
      state,
	  inputMessage,
	  SendMessage,
	  Logout
    }
  }
}
</script>

<style lang="scss">

* {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	margin: 0;
	padding: 0;
	box-sizing: border-box;
	scroll-behavior: smooth;
}
body {
	background-image: linear-gradient(120deg, #84fab0 0%, #8fd3f4 100%);
	//scroll-snap-type: none;
	//overflow: hidden;
}
.view {
	display: flex;
	justify-content: center;
	min-height: 100vh;
	background-image: linear-gradient(120deg, #84fab0 0%, #8fd3f4 100%);
	//opacity: .5;
	
	&.login {
		align-items: center;
		.login-form {
			display: block;
			width: 100%;
			padding: 15px;

			
			@media screen and (min-width: 1000px) {
				width: 40%;
			}

			@media screen and (max-width: 1000px) and (min-width: 500px) {
				width: 70%;;
			}

			
			.form-inner {
				display: block;
				background-color: #FFF;
				padding: 50px 15px;
				border-radius: 16px;
				box-shadow: 0px 6px 12px rgba(0, 0, 0, 0.2);
				h1 {
					color: #AAA;
					font-size: 28px;
					margin-bottom: 30px;
					text-align: center;
				}
				label {
					display: block;
					margin-bottom: 5px;
					color: #AAA;
					font-size: 16px;
					transition: 0.4s;
					text-align: center;
				}
				input[type="text"] {
					text-align: center;
					appearance: none;
					border: none;
					outline: none;
					background: none;
					display: block;
					width: 100%;
					padding: 10px 15px;
					border-radius: 8px;
					margin-bottom: 15px;
					
					color: #333;
					font-size: 18px;
					box-shadow: 0px 0px 0px rgba(0, 0, 0, 0);
					background-color: #F3F3F3;
					transition: 0.4s;
					&::placeholder {
						color: #888;
						transition: 0.4s;
					}
				}
				input[type="submit"] {
					appearance: none;
					border: none;
					outline: none;
					background: none;
					display: block;
					width: 70%;
					margin-left: auto;
					margin-right: auto;
					padding: 10px 15px;
					background-color: #84fab0;
					border-radius: 8px;
					color: #FFF;
					font-size: 18px;
					font-weight: 700;
					cursor: pointer;
				}
				&:focus-within {
					label {
						color: olive;
					}
					input[type="text"] {
						background-color: #FFF;
						box-shadow: 0 0 6px rgba(0, 0, 0, 0.2);
						&::placeholder {
							color: #666;
						}
					}
				}
			}
		}
	}
	&.chat {
		flex-direction: column;
		z-index: 200;
		//overflow: scroll;


		@media screen and (min-width: 1000px) {
			width: 40%;
			margin-left: auto;
			margin-right: auto;
			
		}
		header {
			display: flex;
			justify-content: space-between;
			align-items: center;
			width: 100%;
			padding: 10px 10px 10px;
			height: 60px;
			position: sticky;
			top: 0px;
			background-color: #84fab0;
			filter: brightness(80%);
			border-radius:  8px 8px 0 0;
			z-index: 2;



			.logout {
				right: 15px;
				appearance: none;
				border: none;
				outline: none;
				background: none;
				cursor: pointer;
			}
			h2 {
				color: #333;
			}
		}
		.chat-box {
			//border-radius: 14px 14px 0px 0px;
			background-color: #FFF;
			box-shadow: 0px 0px 12px rgba(100, 100, 100, 0.2);
			flex: 1 1 100%;
			padding: 30px;

			.message {
				display: flex;
				margin-bottom: 15px;
				
				.message-inner {
					.username {
						color: #888;
						font-size: 12px;
						margin-bottom: 5px;
						padding-left: 5px;
						padding-right: 15px;
					}
					.content {
						display: inline-block;
						padding: 10px 20px;
						background-color: lightgray;
						border-radius: 30px;
						color: #333;
						font-size: 14px;
						line-height: 1.2em;
						text-align: left;
					}
				}
				&.current-user {
					margin-top: 30px;
					justify-content: flex-end;
					text-align: right;
					.message-inner {
						max-width: 75%;
						.content {
							color: gray;
							font-weight: 600;
							background-color: #84fab0;
							filter: brightness(80%);
						}
					}
				}
			}
		}
		footer {
			position: sticky;
			bottom: 0px;
			background-color: #FFF;
			padding: 30px;
			box-shadow: 0px 0px 12px rgba(100, 100, 100, 0.2);
			form {
				display: flex;
				input[type="text"] {
					flex: 1 1 100%;
					appearance: none;
					border: none;
					outline: none;
					background: none;
					display: block;
					width: 100%;
					padding: 10px 15px;
					border-radius: 8px 0px 0px 8px;
					
					color: #333;
					font-size: 18px;
					box-shadow: 0px 0px 0px rgba(0, 0, 0, 0);
					background-color: #F3F3F3;
					transition: 0.4s;
					&::placeholder {
						color: #888;
						transition: 0.4s;
					}
				}
				
				button {
					appearance: none;
					border: none;
					outline: none;
					background: none;
					display: block;
					padding: 15px 15px;
					border-radius: 0px 8px 8px 0px;
					background-image: linear-gradient(120deg, #84fab0 0%, #8fd3f4 100%);
					color: #FFF;
					font-size: 18px;
					font-weight: 700;
				}
			}
		}
	}
}
</style>
