<template>
  <!-- <input type="number" v-model="text" /> -->
  <div v-if="startDate != null">กำลังอัด</div>
  <br />
  <br />
  <div class="flex">
    <div style="margin-right: 12px">เริ่มอัดเพลง</div>
    <div>
      <button style="margin-right: 12px" @click="start">start record</button>
      <button style="margin-right: 12px" @click="end">end record</button>
    </div>
  </div>
  <br />
  <br />
  <br />
  <br />
  <br />
  <br />
  <br />
  <div>ลิสต์เพลงเก่า</div>
  <div v-for="(log, i) in allLog">
    <div class="flex">
      <div style="margin-right: 12px">music : {{ i + 1 }}</div>
      <button style="margin-right: 12px" @click="playSong(log, i)">
        play song
      </button>
      <div style="margin-right: 12px" v-if="index === i">กำลังเล่น . . .</div>
    </div>
  </div>
</template>
<style scoped>
.flex {
  display: flex;
}
</style>
<script setup lang="ts">
import { onMounted, onUnmounted, ref } from "vue";
import { keys } from "@/key";
const text = ref(0);
const ctx = new AudioContext();
interface note {
  note: number;
  start: number;
  end: number;
}
interface NoteReturn {
  node: OscillatorNode;
  note: note | null;
}
function ran(min: number, max: number) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}
let log: NoteReturn[] = [];

const startDate: any = ref(null);
const index: any = ref(null);
function playSong(song: NoteReturn[], _index: number) {
  index.value = _index;
  console.log(song);
  song.forEach((n, i) => {
    const { note: _node } = n;
    if (_node) {
      let y = _node.start;
      console.log("startsss", y);
      setTimeout(() => {
        let node = note(_node.note);
        let sttime = _node.end - _node.start;

        console.log("s", sttime);

        setTimeout(() => {
          if (i + 1 === song.length) {
            index.value = null;
          }
          node.node.stop();
        }, sttime);
      }, y);
    }

    // if (i + 1 === song.length) {
    //   setTimeout(() => {
    //     index.value = null;
    //   }, 650);
    // }
  });
}

let stopDate = null;
const allLog = ref<NoteReturn[][]>([]);
function start() {
  startDate.value = new Date();
}

function end() {
  startDate.value = null;
  stopDate = null;
  allLog.value.push(log);
  log = [];
}
let keyList: { key: string; value: number }[] = [];
let coolTime = false;
const scales = [0, 2, 4, 5, 7, 9, 10, 12];
interface playlist {
  key: string;
  node: NoteReturn;
}
let playlist: playlist[] = [];
onMounted(() => {
  let start = 52;
  keys.forEach((x, i) => {
    // console.log(start, i, i % 8, scales[i % 8]);
    // const _key = start + i + scales[i % 8];
    // console.log(_key);

    start++;
    keyList.push({ key: x, value: i });
  });
  keyList = keyList.sort((x, y) => x.value - y.value);
  keyList = keyList.map((x, i) => {
    return {
      key: keys[i],
      value: x.value,
    };
  });
  window.addEventListener("keydown", (e) => {
    // if (!coolTime) {
    if (playlist.some((x) => x.key === e.key)) {
      return;
    }
    const objTostop = note(getNote(e.key));
    playlist.push({ key: e.key, node: objTostop });
    // }
  });
  window.addEventListener("keyup", (e) => {
    const stop = new Date()!;
    console.log(playlist);
    const node = playlist.find((x) => x.key === e.key);
    if (node != undefined) {
      setTimeout(() => {
        if (startDate.value != null) {
          node.node.node.stop();
          node.node.note!.end =
            Math.abs(stop!.getTime() - node.node.note!.end) + 250;
          log.push(node.node);
        } else {
          node.node.node.stop();
        }
      }, 250);
      playlist = playlist.filter((e) => e != node);
    }
    coolTime = false;
  });
});
function getNote(e: string) {
  coolTime = true;

  // return res;

  return keyList.find((x) => x.key == e.toLowerCase())?.value || 1;
}
onUnmounted(() => {
  window.removeEventListener("keydown", () => {});

  window.removeEventListener("keyup", () => {});
});
function note(f: number): NoteReturn {
  //[0,2,4,5,7,9,11,12]
  console.log(f);
  const playTime = 0.1;
  const duration = 0.1;
  const _f = 2 ** ((f - 69) / 12);
  const node = ctx.createOscillator();
  node.type = "triangle";
  const gain = ctx.createGain();
  gain.gain.setValueAtTime(0, ctx.currentTime + playTime);
  gain.gain.linearRampToValueAtTime(
    0.3,
    ctx.currentTime + playTime + duration / 4
  );
  gain.gain.linearRampToValueAtTime(
    0,
    ctx.currentTime + playTime + duration + 1
  );
  const res = 0.07198 * f ** 3 - 1.482 * f ** 2 + 22.26 * f + 40.07;

  node.frequency.value = res;
  gain.connect(ctx.destination);
  node.connect(ctx.destination);
  node.start(ctx.currentTime + playTime);
  const fre = 440 * _f;
  console.log(res);

  // if (startDate.value != null) {
  //   stopDate = new Date();
  //   var timeDiff = Math.abs(stopDate!.getTime() - startDate.value.getTime()); // ใช้ Math.abs() เพื่อให้ได้ค่าที่เป็นบวกเสมอ
  //   var seconds = timeDiff / 1000; // เปลี่ยนจากมิลลิวินาทีเป็นวินาที
  // } else {
  //   console.log("recode", f);
  // }

  if (startDate.value != null) {
    return {
      node,
      note: {
        note: f,
        start: Math.abs(new Date()!.getTime() - startDate.value.getTime()),
        end: startDate.value.getTime(),
      },
    };
  } else {
    return { node, note: null };
  }
  // return [
  //   node,
  //   {
  //     note: f,
  //     start: Math.abs(new Date()!.getTime() - startDate.value.getTime()),
  //     end: null,
  //   },
  // ];
  // node.stop(ctx.currentTime + playTime + duration);
}
</script>
