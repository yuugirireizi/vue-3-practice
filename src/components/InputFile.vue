<script setup lang="ts">
import {ref} from 'vue';

type Bitmap = {
  isBitmap: boolean;
  fileSize: number;
  offsetToPixelData: number;
  width: number;
  height: number;
}

const input = ref(null as HTMLInputElement | null);
const imageUrl = ref("");
const bitmap = ref(null as Bitmap | null);

function change(): void {
  const files = input.value?.files;
  if (files == null) {
    return;
  }

  for (let i = 0; i < files.length; i++) {
    const file = files[i];
    const reader = new FileReader();
    reader.onload = (event: ProgressEvent<FileReader>): void => {
      const fileBinary = event.target?.result;
      if (fileBinary == null) {
        return;
      }

      if (typeof fileBinary === 'string') {
        return;
      }

      // BMPを読み込む
      /* BMPファイルフォーマットは下記サイトを参照した。
       * https://www.setsuki.com/hsp/ext/bmp.htm
       */
      const fileHeader = new DataView(fileBinary.slice(0, 14));
      const wordB = fileHeader.getUint8(0);
      const wordM = fileHeader.getUint8(1);
      const isBitmap = wordB === 0x42 && wordM === 0x4d;
      const fileSize = fileHeader.getUint32(2, true);
      // 予約領域は無視する。
      const offsetToPixelData = fileHeader.getUint32(10, true);

      const infoHeader = new DataView(fileBinary.slice(14, offsetToPixelData));
      const infoHeaderSize = infoHeader.getUint32(0, true);
      if (infoHeaderSize !== 40) {
        return;
      }
      const width = infoHeader.getUint32(4, true);
      const height = infoHeader.getUint32(8, true);

      bitmap.value = {
        isBitmap,
        fileSize,
        offsetToPixelData,
        width,
        height
      }

      // URLに変換して画像表示
      imageUrl.value = URL.createObjectURL(new Blob([fileBinary], {type: 'image/bmp'}));
    }
    reader.readAsArrayBuffer(file);
  }
}
</script>

<template>
  <div>
    <input type="file" ref="input" @change="change">
    <div>
      <div v-if="bitmap != null">
        <div>
          <span v-if="bitmap.isBitmap">ファイルはBMP画像です。</span>
          <span v-else>ファイルはBMP画像ではありません。</span>
        </div>
        <div>
          <span>ファイルサイズ: {{ bitmap.fileSize }}</span>
        </div>
        <div>
          <span>画像データまでのオフセット: {{ bitmap.offsetToPixelData }}</span>
        </div>
        <div>
          <span>幅: {{ bitmap.width }}、高さ: {{ bitmap.height }}</span>
        </div>
        <img :src="imageUrl"/>
      </div>
    </div>
  </div>
</template>

<style scoped>

</style>