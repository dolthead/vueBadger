<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar color="primary">
        <ion-title>Vue PWA Notifications!</ion-title>
        <ion-badge slot="end" color="danger">{{ badgeCount }}</ion-badge>
      </ion-toolbar>
    </ion-header>

    <ion-content>

      <ion-list lines="none">

        <ion-item>
          <ion-input label="Message" label-placement="floating" v-model="message" :clear-input="true" @keyup.enter="notify"></ion-input>
        </ion-item>
        <ion-button expand="block" :disabled="!supported" @click="notify" size="default">
          Send Message
        </ion-button>

        <ion-item-divider />

        <ion-item class="inbox">
          <ion-label>Inbox</ion-label>
          <ion-button slot="end" size="small" fill="outline" @click="markAllRead()">Mark all as read</ion-button>
        </ion-item>

        <ion-item v-for="message of messageList" :key="message.id" class="message" :class="{ 'read' : message.read }" button @click="markRead(message)">
          <ion-label>{{ message.message }}</ion-label>
          <ion-fab-button slot="end" size="small" color="white" fill="none" @click.stop="deleteMessage(message.id)">
            <ion-icon size="small" color="danger" :icon="trash"></ion-icon>
          </ion-fab-button>
        </ion-item>
      </ion-list>

    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { IonContent, IonHeader, IonPage, IonTitle, IonBadge, IonToolbar, IonButton, IonFabButton, IonInput, IonLabel, IonItemDivider, IonList, IonItem, IonIcon, toastController, onIonViewDidEnter } from '@ionic/vue';
import { UseWebNotificationOptions, useWebNotification } from '@vueuse/core'
import { trash } from 'ionicons/icons';
import { ref } from 'vue';

const message = ref('Hello World!');
let notificationCount = 0;
const messageList = ref([
  { id: 1, message: 'Hello World!', read: false },
  { id: 2, message: 'Hello World!', read: false },
  { id: 3, message: 'Hello World!', read: false },
  { id: 4, message: 'Hello World!', read: true },
]);
let nextId = 5;
const badgeCount = ref(0);

const options = () => ({
  title: message.value,
  body: message.value,
  dir: 'auto',
  lang: 'en',
  renotify: true,
  // tag: 'vue-pwa-notification', // resend with same tag to replace previous notification
  tag: `tag-${notificationCount++}`, // resend with unique tag to queue up many unique notifications
  icon: "/pwa-192x192.png",
  vibrate: [200, 100, 200],
} as UseWebNotificationOptions);
let supported = false;
let swRegistration: ServiceWorkerRegistration;

onIonViewDidEnter(() => {
  const { isSupported, close, onClick, onShow } = useWebNotification(options());
  supported = isSupported.value;

  updateAppBadge();

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

  navigator.serviceWorker.ready
      .then((registration) => swRegistration = registration, console.error);
});

const deleteMessage = (id:number) => {
  const index = messageList.value.findIndex(message => message.id === id);
  if (index > -1) {
    messageList.value.splice(index, 1);
  }
  updateAppBadge();
};

const markRead = (message: any) => {
  message.read = true;
  updateAppBadge();
};

const markAllRead = () => {
  messageList.value.forEach(message => message.read = true);
  updateAppBadge();
};

const updateAppBadge = async () => {
  badgeCount.value = messageList.value.filter(message => !message.read).length;
  if (badgeCount.value === 0) {
    navigator.clearAppBadge && await navigator.clearAppBadge();
  } else {
    navigator.setAppBadge && await navigator.setAppBadge(badgeCount.value)
  }
};

const notify = () => {
  messageList.value.unshift({ id: nextId++, message: message.value, read: false });
  updateAppBadge();
  const { show } = useWebNotification(options());
  if (swRegistration) {
    swRegistration.showNotification("Vue Badger", options());
    updateAppBadge();
  } else {
    show();
  }
};
</script>

<style scoped>
ion-badge {
  border-radius: 9999px;
  margin-top: -8px;
  min-width: 20px;
  padding: 8px;
  margin-inline-end: 16px;
}
ion-button {
  min-width: 130px;
  margin-inline: 12px;
}
ion-item-divider {
  border-bottom: none;
  background-color: white;
}
.message {
  & ::part(native) {
    min-height: 32px !important;
  }
  & ion-label {
    font-weight: bold;
  }
}
.message.read ion-label {
  color: gray;
  font-weight: normal;
}
ion-fab-button {
  --background: transparent;
  --box-shadow: none;
}
.inbox ion-label {
  font-weight: bold;
  font-size: x-large;
}
</style>
