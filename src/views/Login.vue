<template>
  <ion-page >
    <ion-header>
      <ion-toolbar>
        <ion-title>Login</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true" class="ion-padding">
      <div class="container">
        <ion-list class="list-container">
          <ion-item>
            <ion-label position="floating">Phone</ion-label>
            <ion-input v-model="telemovel" type="number" maxlength="9"></ion-input>
          </ion-item>
          <ion-item>
            <ion-label position="floating">Password</ion-label>
            <ion-input v-model="password" type="password"></ion-input>
          </ion-item>
          <ion-item>
            <ion-label position="floating">PIN</ion-label>
            <ion-input v-model="pin" type="password"></ion-input>
          </ion-item>
        </ion-list>
        <div v-show="errors.length > 0">
          <p v-for="error in errors" class="error">{{ error }}</p>
        </div>
        <ion-button expand="block" @click="login">Create</ion-button>
      </div>
    </ion-content>
  </ion-page>
</template>

  
  <script setup>
  import { ref, inject, onBeforeMount } from 'vue';
  import { useRouter } from 'vue-router';
  import { Storage } from '@ionic/storage';
  import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonList, IonItem, IonLabel, IonInput, IonButton, onIonViewDidEnter, onIonViewWillEnter } from '@ionic/vue';
  
  const axios = inject('axios');
  const router = useRouter();

  const store = new Storage();

  onIonViewWillEnter(async () => {
    await store.create();

    const token = await store.get('token')
    if (token){
      router.push('/dashboard');
    }

  });

  const telemovel = ref('');
  const password = ref('');
  const pin = ref('');


  const errors = ref([]);




import { FCM } from "@capacitor-community/fcm";
import { PushNotifications } from '@capacitor/push-notifications';
// external required step
// register for push




const register = async (phone_number) => {
    // await PushNotifications.requestPermissions();
    // await PushNotifications.register();

    PushNotifications.requestPermissions().then(result => {
        console.log('result ' + JSON.stringify(result));
        // alert('result ' + JSON.stringify(result));
        // Initialize the registration with FCM Token
        // FCM.init(result);
        PushNotifications.register();
        
      });
      // On registration we can get the token
      PushNotifications.addListener('registration', (token) => {
        console.log('Push registration success, token: ' + token.value);
        // alert('Push registration success, token: ' + token.value);
        //TODO: store token server side?
      });


    // now you can subscribe to a specific topic
    FCM.subscribeTo({ topic: `${phone_number}` })
      .then((r) => console.log(`subscribed to topic`))
      .catch((err) => console.log(err));

    // // Unsubscribe from a specific topic
    // // FCM.unsubscribeFrom({ topic: "test" })
    // //   .then(() => alert(`unsubscribed from topic`))
    // //   .catch((err) => console.log(err));

    // // Get FCM token instead of the APN one returned by Capacitor
    // FCM.getToken()
    //   .then((r) => alert(`Token ${r.token}`))
    //   .catch((err) => console.log(err));

    // Delete the old FCM token and get a new one
    // FCM.refreshToken()
    //   .then((r) => alert(`Token ${r.token}`))
    //   .catch((err) => console.log(err));

    // Remove FCM instance
    // FCM.deleteInstance()
    //   .then(() => alert(`Token deleted`))
    //   .catch((err) => console.log(err));

    // Enable the auto initialization of the library
    // FCM.setAutoInit({ enabled: true }).then(() => alert(`Auto init enabled`));

    // // Check the auto initialization status
    // FCM.isAutoInitEnabled().then((r) => {
    //   console.log("Auto init is " + (r.enabled ? "enabled" : "disabled"));
    // });
    }







  const login = () => {
      // return router.push('/dashboard');
      //valida se o campo telemovel está vazio e se tem 9 digitos e começa por 9
      errors.value = [];
      let valid = true;
      if (telemovel.value == '' || telemovel.value.length != 9 || telemovel.value.charAt(0) != 9) {
          errors.value.push('The phone field is empty or does not have 9 digits or does not start with 9');
          valid = false;
      }
      //valida se o campo password está vazio
      if (password.value == '') {
        errors.value.push('The password field is empty');
        valid = false;
      }

      //valida se o campo pin está vazio e se tem 4 digitos
      if (pin.value == '' || pin.value.length != 3) {
        errors.value.push('The pin field is empty or does not have 3 digits');
        valid = false;
      }


      if (valid){
        axios.post('/auth/login', {
          username: telemovel.value,
          password: password.value,
          confirmation_code: pin.value
        }).then(async (response) => {
          if (response.data.access_token){

            try{
              await store.set('token', response.data.access_token);
              await store.set('phone_number', telemovel.value);
              await store.set('pin', pin.value);
              
              if(Capacitor.getPlatform() === 'web') {
                console.log('PushNotifications not supported on web');
              } else {
                await register(telemovel.value);
              }
              
              await store.set('autosavings', true);
              await store.set('togglenotifications', true);
              router.push('/dashboard');
              
              telemovel.value = '';
              password.value = '';
              pin.value = '';
            } catch (error){
              console.log(error);
            }
          }
        }, (error) => {
          console.log(error);
          if (error.response.status == 401 || error.response.status == 400){
            errors.value.push('Invalid credentials');
          }
        });
      }
  };
  </script>
  
  <style scoped>
  .container {
    display: flex;
    flex-direction: column;
    height: 100%;
    justify-content: center;
    align-items: center;
  }
  
  .list-container {
    width: 100%;
    max-width: 400px; /* Adjust max-width as needed */
  }
  
  .error {
    color: red;
    text-align: center;
  }
  </style>