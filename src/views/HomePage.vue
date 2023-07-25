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
        <ion-item>
          <ion-input label="Message" label-placement="floating" v-model="message" :clear-input="true"></ion-input>
          <ion-button slot="end" expand="block" :disabled="!supported" @click="notify($event)" size="default">Notify
            me</ion-button>
        </ion-item>
        <ion-item>
          <ion-input type="number" label-placement="floating" label="Badge count" v-model="badgeCount"></ion-input>
          <ion-button slot="end" expand="block" :disabled="!supported" @click="badgeCount = 0" size="default">
            Clear
            <ion-badge v-if="badgeCount" color="danger">{{ badgeCount }}</ion-badge>
          </ion-button>
        </ion-item>
      </ion-list>

    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonButton, IonInput, IonBadge, IonList, IonItem, toastController, onIonViewDidEnter } from '@ionic/vue';
import { UseWebNotificationOptions, useWebNotification } from '@vueuse/core'
import { ref, watch } from 'vue';

const message = ref('Hello World!');
const notificationCount = ref(0);
const badgeCount = ref(3);

const options = () => ({
  title: message.value,
  body: message.value,
  dir: 'auto',
  lang: 'en',
  renotify: true,
  tag: `tag-${notificationCount.value++}`,
  icon: "/pwa-192x192.png",
  vibrate: [200, 100, 200],
  // badge: '/red-dot.png',
  // actions: [ { action: 'Close', title: 'Close' }],
} as UseWebNotificationOptions);
let supported = false;

onIonViewDidEnter(() => {
  const { isSupported, close, onClick, onShow } = useWebNotification(options());
  supported = isSupported.value;

  onShow(event => {
    console.log('Notification shown', event);
    toastController.create({
      message: 'Notification shown',
      duration: 3000,
    }).then(toast => toast.present());
  });

  onClick(event => {
    console.log('Notification clicked', event);
    close();
    toastController.create({
      message: 'Notification clicked',
      duration: 3000,
    }).then(toast => toast.present());
  });
  
});

const updateAppBadge = async () => {
  if (badgeCount.value === 0) {
    navigator.clearAppBadge && await navigator.clearAppBadge();
  } else {
    navigator.setAppBadge && await navigator.setAppBadge(badgeCount.value)
  }
};

watch(badgeCount, updateAppBadge);

const notify = async (event: any) => {
  const { show } = useWebNotification(options());
  try {
    await show(event);
  } catch (error) {
    console.log(error);
    // for chrome android
    navigator.serviceWorker.ready
      .then((registration) => {
        registration.showNotification("Vue Badger", options());
        updateAppBadge();
      });
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
