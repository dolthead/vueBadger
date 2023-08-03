<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar color="primary">
        <ion-title>Vue PWA Notifications!</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>

      <ion-list lines="none" class="zoom">

        <ion-item>
          <ion-input label="Message" label-placement="floating" v-model="message" :clear-input="true"></ion-input>
        </ion-item>
        <ion-button expand="block" :disabled="!supported" @click="notify" size="default">
          Send Message
        </ion-button>
        <ion-item-divider />
        <ion-item>
          <ion-fab-button slot="start" size="small" @click="updateBadgeCount(--badgeCount)">
            <ion-icon :icon="remove"></ion-icon>
          </ion-fab-button>
          <ion-input class="elbow-room" type="number" label-placement="floating" label="Badge count" v-model="badgeCount"></ion-input>
          <ion-fab-button slot="end" size="small" @click="updateBadgeCount(++badgeCount)">
            <ion-icon :icon="add"></ion-icon>
          </ion-fab-button>
        </ion-item>
        <ion-button expand="block" :disabled="!supported || !badgeCount" @click="updateBadgeCount(0)" size="default">
          Clear, Mark All Read
          <ion-badge v-if="badgeCount" color="danger">{{ badgeCount }}</ion-badge>
        </ion-button>
        
        <!-- <ion-item v-for="message of messageList" :key="message.id" :class="{ 'read' : message.read }">
          <ion-icon icon="notifications" slot="start"></ion-icon>
          <ion-label>{{ message.message }}</ion-label>
        </ion-item> -->
      </ion-list>

    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonButton, IonInput, IonBadge, IonFabButton, IonItemDivider, IonList, IonItem, IonIcon, toastController, onIonViewDidEnter } from '@ionic/vue';
import { UseWebNotificationOptions, useWebNotification } from '@vueuse/core'
import { add, remove } from 'ionicons/icons';
import { ref, watch } from 'vue';

const message = ref('Hello World!');
const notificationCount = ref(0);
const badgeCount = ref(3);
// const messageList = ref([
//   { id: 1, message: 'Hello World!', read: false },
//   { id: 2, message: 'Hello World!', read: false },
//   { id: 3, message: 'Hello World!', read: false },
//   { id: 4, message: 'Hello World!', read: true },
// ]);
// let nextId = 5;

const options = () => ({
  title: message.value,
  body: message.value,
  dir: 'auto',
  lang: 'en',
  renotify: true,
  tag: `tag-${notificationCount.value++}`,
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

const updateBadgeCount = async (x: number) => {
  badgeCount.value = x;
  if (badgeCount.value < 0) {
    badgeCount.value = 0;
  }
};

const updateAppBadge = async () => {
  if (badgeCount.value === 0) {
    navigator.clearAppBadge && await navigator.clearAppBadge();
  } else {
    navigator.setAppBadge && await navigator.setAppBadge(badgeCount.value)
  }
};

watch(badgeCount, updateAppBadge);

const notify = () => {
  updateBadgeCount(++badgeCount.value);
  // messageList.value.unshift({ id: nextId++, message: message.value, read: false });
  // badgeCount.value++;
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
  margin-inline: 12px;
}

ion-item-divider {
  border-bottom: none;
  background-color: white;
}

.zoom {
  zoom: 1.4;
}
.read {
  color: gray;
}
</style>
