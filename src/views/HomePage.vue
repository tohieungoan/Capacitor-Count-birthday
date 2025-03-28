<template>
  <ion-page>
    <ion-header>
      <ion-toolbar color="primary">
        <ion-title>🎂 Đếm Ngược Sinh Nhật 🎉</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding custom-bg">
      <div class="container">
        <h2 class="title">🎈 Nhập ngày sinh của bạn 🎈</h2>

        <!-- Input ngày sinh -->
        <ion-item class="custom-input">
          <ion-label position="floating">📅 Ngày sinh (dd/mm)</ion-label>
          <ion-input v-model="birthday" type="text" placeholder="VD: 25/12" />
        </ion-item>

        <!-- Nút tính toán -->
        <ion-button expand="full" color="tertiary" @click="calculateDaysLeft">
          🚀 Tính toán
        </ion-button>

        <!-- Hiển thị kết quả -->
        <ion-card v-if="daysLeft !== null" class="custom-card">
          <ion-card-content>
            <h3>🎂 Sinh nhật bạn còn <b>{{ daysLeft }}</b> ngày nữa! 🎊</h3>
          </ion-card-content>
        </ion-card>

        <!-- Nút chia sẻ -->
        <ion-button v-if="daysLeft !== null" expand="full" color="success" @click="shareCountdown">
          📢 Chia sẻ ngay!
        </ion-button>

        <!-- Hiển thị mức pin -->
        <ion-card v-if="batteryLevel !== null" class="battery-card">
          <ion-card-content>
            🔋 Mức pin: <b>{{ batteryLevel === -1 ? "Không khả dụng" : batteryLevel + "%" }}</b>
          </ion-card-content>
        </ion-card>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { 
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent, 
  IonItem, IonLabel, IonInput, IonButton, IonCard, IonCardContent 
} from '@ionic/vue';
import { LocalNotifications } from '@capacitor/local-notifications';
import { Share } from '@capacitor/share';
import { Device } from '@capacitor/device';

// Biến lưu ngày sinh, số ngày còn lại và mức pin
const birthday = ref<string>('');
const daysLeft = ref<number | null>(null);
const batteryLevel = ref<number | null>(null);

// Cấp quyền thông báo và lấy mức pin khi app khởi động
onMounted(() => {
  LocalNotifications.requestPermissions();
  getBatteryStatus();
});

// Hàm tính số ngày còn lại đến sinh nhật
const calculateDaysLeft = async () => {
  if (!birthday.value.match(/^\d{2}\/\d{2}$/)) {
    alert("⚠️ Vui lòng nhập đúng định dạng dd/mm!");
    return;
  }

  const [day, month] = birthday.value.split('/').map(Number);
  const today = new Date();
  const currentYear = today.getFullYear();

  // Xác định sinh nhật năm nay hoặc năm sau
  let birthdayDate = new Date(currentYear, month - 1, day);
  if (birthdayDate < today) {
    birthdayDate.setFullYear(currentYear + 1);
  }

  // Tính số ngày còn lại
  const diffTime = birthdayDate.getTime() - today.getTime();
  daysLeft.value = Math.ceil(diffTime / (1000 * 60 * 60 * 24));

  // Gửi thông báo
  await sendNotification();
};

// Hàm gửi thông báo cục bộ
const sendNotification = async () => {
  if (daysLeft.value !== null) {
    await LocalNotifications.schedule({
      notifications: [
        {
          title: "🎂 Đếm Ngược Sinh Nhật",
          body: `📆 Chỉ còn ${daysLeft.value} ngày nữa đến sinh nhật bạn! 🎉`,
          id: 1,
          schedule: { at: new Date(Date.now() + 1000) }, // Hiển thị ngay lập tức
        },
      ],
    });
  }
};

// Hàm chia sẻ kết quả đếm ngược
const shareCountdown = async () => {
  if (daysLeft.value !== null) {
    await Share.share({
      title: "🎂 Đếm Ngược Sinh Nhật 🎉",
      text: `Sinh nhật của tôi còn ${daysLeft.value} ngày nữa! 🎈`,
      url: "https://www.facebook.com/hieungoan.to.1", // Thay bằng link của ứng dụng
      dialogTitle: "Chia sẻ qua",
    });
  }
};

// Hàm lấy trạng thái pin
const getBatteryStatus = async () => {
  try {
    const info = await Device.getBatteryInfo();
    if (info.batteryLevel !== undefined && info.batteryLevel !== null) {
      batteryLevel.value = Math.round(info.batteryLevel * 100);
    } else {
      batteryLevel.value = -1; // Gán giá trị mặc định nếu không lấy được
    }
  } catch (error) {
    console.error("Không thể lấy thông tin pin:", error);
    batteryLevel.value = -1; // Gán giá trị mặc định để tránh lỗi
  }
};
</script>

<style scoped>
/* Nền gradient */
.custom-bg {
  background: linear-gradient(to bottom, #ff9a9e, #fad0c4);
}

/* Tiêu đề */
.title {
  font-size: 22px;
  font-weight: bold;
  color: #7434eb;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.2);
}

/* Container căn giữa */
.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
  gap: 15px;
  margin-top: 20px;
}

/* Input ngày sinh */
.custom-input {
  width: 90%;
  max-width: 400px;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 10px;
}

/* Card kết quả */
.custom-card {
  background: linear-gradient(to right, #ff9966, #ff5e62);
  color: white;
  border-radius: 15px;
  text-align: center;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.2);
}

/* Nút */
ion-button {
  width: 90%;
  max-width: 400px;
  font-size: 18px;
  border-radius: 50px;
}

/* Card pin */
.battery-card {
  background: linear-gradient(to right, #5ee7df, #b490ca);
  color: white;
  text-align: center;
  border-radius: 15px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.2);
}
</style>
