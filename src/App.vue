<script setup>
import {
  Room,
  RoomEvent,
  Track,
  createLocalAudioTrack,
  createLocalVideoTrack,
} from 'livekit-client';
import { onMounted, onUnmounted, ref } from 'vue';

const room = ref(null);
const url = ref('');
const token = ref('');
const parentElement = ref(null);

onMounted(() => {
  room.value = new Room();
});

onUnmounted(() => {
  if (room.value) {
    room.value.disconnect();
  }
});

function handleTrackSubscribed(track, publication, participant) {
  console.log("收到轨道:", track.kind);
  
  if (track.kind === Track.Kind.Video || track.kind === Track.Kind.Audio) {
    const element = track.attach();
    
    if (track.kind === Track.Kind.Video) {
      element.style.width = '300px';
      element.style.height = '200px';
      element.style.margin = '10px';
      element.style.border = '2px solid #ccc';
    }
    
    if (parentElement.value) {
      parentElement.value.appendChild(element);
    }
  }
}

async function connect() {
  if (!url.value || !token.value) {
    alert('请输入 URL 和 Token');
    return;
  }

  try {
    // 监听轨道订阅事件
    room.value.on(RoomEvent.TrackSubscribed, handleTrackSubscribed);
    
    // 连接房间
    await room.value.connect(url.value, token.value);
    console.log('成功连接到房间');
    
    // 创建并发布本地视频轨道
    const videoTrack = await createLocalVideoTrack();
    await room.value.localParticipant.publishTrack(videoTrack);
    
    // 显示本地视频
    const videoElement = videoTrack.attach();
    videoElement.style.width = '300px';
    videoElement.style.height = '200px';
    videoElement.style.margin = '10px';
    videoElement.style.border = '2px solid #00ff00';
    videoElement.muted = true; // 本地视频静音
    
    if (parentElement.value) {
      parentElement.value.appendChild(videoElement);
    }
    
    // 创建并发布本地音频轨道
    const audioTrack = await createLocalAudioTrack();
    await room.value.localParticipant.publishTrack(audioTrack);
    
    console.log('音视频轨道已发布');
    
  } catch (error) {
    console.error('连接失败:', error);
  }
}

function disconnect() {
  if (room.value) {
    room.value.disconnect();
    // 清空视频容器
    if (parentElement.value) {
      parentElement.value.innerHTML = '';
    }
  }
}
</script>

<template>
  <div>
    <div style="margin-bottom: 20px;">
      <input 
        type="text" 
        v-model="url" 
        placeholder="LiveKit URL"
        style="margin-right: 10px; padding: 8px; width: 300px;"
      >
      <input 
        type="text" 
        v-model="token" 
        placeholder="Token"
        style="margin-right: 10px; padding: 8px; width: 300px;"
      >
      <button @click="connect" style="margin-right: 10px; padding: 8px 15px;">
        加入房间
      </button>
      <button @click="disconnect" style="padding: 8px 15px;">
        离开房间
      </button>
    </div>

    <div 
      ref="parentElement" 
      style="width: 100%; height: 500px; background: #000; border: 1px solid #ccc; display: flex; flex-wrap: wrap; align-items: flex-start; padding: 10px;">
    </div>
  </div>
</template>