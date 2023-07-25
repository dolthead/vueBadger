<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar color="primary">
        <ion-title>Vue PWA Notify Me</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>

      <br>
      <ion-list lines="none">
        <!-- <ion-item>
          <ion-button expand="block" :disabled="!isSupported" @click="getPermission" size="default">Request permission</ion-button>
        </ion-item> -->
        <ion-item>
          <ion-input label="Message" label-placement="floating" v-model="message" :clear-input="true"></ion-input>
          <ion-button slot="end" expand="block" :disabled="!isSupported" @click="notify" size="default">Notify
            me</ion-button>
        </ion-item>
        <ion-item>
          <ion-input type="number" label-placement="floating" label="Badge count" v-model="badgeCount"></ion-input>
          <ion-button slot="end" expand="block" :disabled="!isSupported" @click="badgeCount = 0" size="default">
            Clear
            <ion-badge v-if="badgeCount" color="danger">{{ badgeCount }}</ion-badge>
          </ion-button>
        </ion-item>
      </ion-list>
      <br>
      <br>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonButton, IonInput, IonBadge, IonList, IonItem, onIonViewDidEnter, toastController } from '@ionic/vue';
import { UseWebNotificationOptions, useWebNotification } from '@vueuse/core'
import { ref, watch } from 'vue';

const message = ref('Hello World!');
const notificationCount = ref(0);
const options = {
  title: message.value,
  dir: 'auto',
  lang: 'en',
  renotify: true,
  tag: `tag-${notificationCount.value++}`,
  vibrate: [200, 100, 200],
} as UseWebNotificationOptions;
const {
  isSupported,
  notification,
  show,
  close,
  onClick,
  onShow,
  onError,
  onClose,
} = useWebNotification(options);

onClick(event => {
  console.log('Notification clicked', event);
  toastController.create({
    message: 'Notification clicked',
    duration: 3000,
  }).then(toast => toast.present());
});

const badgeCount = ref(3);
const updateAppBadge = () => {
  if (badgeCount.value === 0) {
    navigator.clearAppBadge();
  } else {
    navigator.setAppBadge(badgeCount.value)
  }
};
onIonViewDidEnter(() => {
  updateAppBadge();
});

watch(badgeCount, updateAppBadge);

const getPermission = (event: MouseEvent) => {
  if (navigator.serviceWorker && window.PushManager && window.Notification) {

    // navigator.serviceWorker.ready
    // .then(registration => {
    //     registration.pushManager.getSubscription()
    //     .then(pushSubscription => {
    //         if(!pushSubscription){
    //             //the user was never subscribed
    //             pushSubscription!.subscribe(registration);
    //         }
    //         else{
    //             //check if user was subscribed with a different key
    //             let json = pushSubscription.toJSON();
    //             let public_key = json.keys.p256dh;
                
    //             console.log(public_key);
                
    //             if(public_key != NEW_PUBLIC_KEY){
    //                 pushSubscription.unsubscribe().then(successful => {
    //                     // You've successfully unsubscribed
    //                     registration.pushManager.subscribe(registration);
    //                 }).catch(e => {
    //                     // Unsubscription failed
    //                 })
    //             }
    //         }
    //     });
    // })

    // Request permission to send push notifications
    // navigator.serviceWorker.getRegistration()
    //   .then(function(registration) {
    //     registration!.pushManager.subscribe({ userVisibleOnly: true })
    //       .then(function(subscription) {
    //         console.log('Push notifications are allowed.');
    //         //save the push subscription in your database
    //       }).catch(function(error) {
    //         console.log('Error:', error);
    //       });
    //   });
  }
  // try {
  //   const result = await Notification.requestPermission();
  //   console.log(result);
  //   if (result === 'granted') {
  //     show();
  //   }
  // } catch (e) {
  //   console.error(e);
  // }
};

const notify = async () => {
  try {
    await show();
  } catch (e1) {
    try {
      notification.value = new Notification(message.value, {
        ...options,
        tag: `tag-${notificationCount.value++}`,
      });
    } catch (e2) {
      // for chrome android
      navigator.serviceWorker.ready.then((registration) => {
        registration.showNotification("Vue Badger", {
          actions: [
            {
              action: "close",
              title: "Close",
              icon: "/pwa-192x192.png",
            },
            {
              action: "open",
              title: "Open",
              icon: "/pwa-192x192.png",
            },
          ],
          body: message.value,
          icon: "/pwa-192x192.png",
          vibrate: [200, 100, 200],
          badge: `${badgeCount.value}`,
          tag: `tag-${notificationCount.value}`,
        });
      });
    }
  }
};
</script>

<style scoped>
ion-badge {
  --background: purple;
  --color: white;
  margin-left: 4px;
  border-radius: 9999px;
  margin-top: -8px;
  min-width: 20px;
  scale: 0.8;
}
ion-button {
  min-width: 130px;
  margin-top: 16px;
}
</style>
