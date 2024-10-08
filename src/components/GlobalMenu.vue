<script setup lang="ts">
import { onClickOutside } from "@vueuse/core";
import { computed, ref } from "vue";
import { getYouTubeVideoId } from "@/libs/youtube";
import { usePlayerStore } from "@/stores/playerStore";
import { useVideoListStore } from "@/stores/videoListStore";

const playerStore = usePlayerStore();
const videoListStore = useVideoListStore();

const urlInputTextarea = ref("");
// メニューの開閉
const isOpen = ref(false);

const menuEl = ref<HTMLElement | null>(null);
const editListPopoverEl = ref<HTMLElement | null>(null);

const hasVideos = computed(() => videoListStore.videoIdList.length > 0);

// 動画追加フォームのハンドリング
function onUrlsSubmit(e: Event) {
  e.preventDefault();
  const urls = urlInputTextarea.value
    .split("\n")
    .map((line) => line.trim())
    .filter((line) => line.length > 0);

  // 入力クリア
  urlInputTextarea.value = "";
  editListPopoverEl.value?.hidePopover();

  const vids = urls.flatMap((url) => getYouTubeVideoId(url) ?? []);
  if (vids.length === 0) return;
  videoListStore.addVideoList(vids);
}

function toggleOpen() {
  isOpen.value = !isOpen.value;
  if (!isOpen.value) return;

  // 外をクリックしたら閉じる
  const removeClickOutsideListener = onClickOutside(menuEl, () => {
    isOpen.value = false;
    removeClickOutsideListener();
  });
}
</script>

<template>
  <div
    ref="menuEl"
    class="absolute right-2 top-2 z-10 flex w-fit flex-col items-end justify-center bg-yellow-400 shadow-md outline"
    :class="`${isOpen ? 'rounded-xl' : 'rounded-full'}`"
  >
    <div class="flex flex-row">
      <div v-if="isOpen" class="flex flex-row">
        <button
          :disabled="!hasVideos"
          class="grid size-11 place-items-center rounded-full hover:bg-yellow-200 disabled:opacity-20"
          @click="playerStore.playAll"
          title="Play all"
        >
          <i class="i-mdi-play size-8" />
        </button>

        <button
          :disabled="!hasVideos"
          class="grid size-11 place-items-center rounded-full hover:bg-yellow-200 disabled:opacity-20"
          @click="playerStore.pauseAll"
          title="Pause all"
        >
          <i class="i-mdi-pause size-8" />
        </button>

        <button
          :disabled="!hasVideos"
          class="grid size-11 place-items-center rounded-full hover:bg-yellow-200 disabled:opacity-20"
          @click="playerStore.muteAll"
          title="Mute all"
        >
          <i class="i-mdi-volume-mute size-8" />
        </button>
      </div>

      <button
        class="grid size-11 place-items-center rounded-full hover:bg-yellow-200"
        @click="toggleOpen"
        title="Menu"
      >
        <i class="i-mdi-dots-vertical size-8" />
      </button>
    </div>
    <div v-if="isOpen" class="flex flex-col items-end justify-center">
      <div class="flex items-center">
        <button
          :disabled="!hasVideos"
          class="relative grid size-11 place-items-center rounded-full hover:bg-yellow-200 disabled:opacity-20"
          title="Seek all -10m"
          @click="playerStore.seekOffsetAll(-10 * 60)"
        >
          <i class="i-mdi-rewind size-7" />
          <p class="absolute bottom-0 text-xs font-bold">-10m</p>
        </button>
        <button
          :disabled="!hasVideos"
          class="relative grid size-11 place-items-center rounded-full hover:bg-yellow-200 disabled:opacity-20"
          title="Seek all -1m"
          @click="playerStore.seekOffsetAll(-60)"
        >
          <i class="i-mdi-rewind size-6" />
          <p class="absolute bottom-0 text-xs font-bold">-1m</p>
        </button>
        <button
          :disabled="!hasVideos"
          class="relative grid size-11 place-items-center rounded-full hover:bg-yellow-200 disabled:opacity-20"
          title="Seek all +1m"
          @click="playerStore.seekOffsetAll(60)"
        >
          <i class="i-mdi-fast-forward size-6" />
          <p class="absolute bottom-0 text-xs font-bold">+1m</p>
        </button>
        <button
          :disabled="!hasVideos"
          class="relative grid size-11 place-items-center rounded-full hover:bg-yellow-200 disabled:opacity-20"
          title="Seek all +10m"
          @click="playerStore.seekOffsetAll(10 * 60)"
        >
          <i class="i-mdi-fast-forward size-7" />
          <p class="absolute bottom-0 text-xs font-bold">+10m</p>
        </button>
      </div>
      <!-- <div class="flex items-center">
        連結してシーク
        <button
          :disabled="!hasVideos"
          class="relative grid size-11 place-items-center rounded-full hover:bg-yellow-200 disabled:opacity-20"
          @click="playerStore.toggleSyncSeek"
          title="Sync seek"
        >
          <i
            class="size-8"
            :class="`${playerStore.isSyncSeek ? 'i-mdi-toggle-switch' : 'i-mdi-toggle-switch-off-outline'}`"
          />
          <p class="absolute bottom-0 text-xs font-bold">
            {{ playerStore.isSyncSeek ? "ON" : "OFF" }}
          </p>
        </button>
      </div> -->
      <div class="flex items-center">
        動画を追加
        <button
          class="grid size-11 place-items-center rounded-full hover:bg-yellow-200"
          popovertarget="editListPopover"
          title="Add video"
        >
          <i class="i-mdi-add size-8" />
        </button>

        <div
          popover
          ref="editListPopoverEl"
          id="editListPopover"
          class="bottom-auto left-auto right-2 top-2 overflow-visible bg-transparent p-0"
        >
          <div class="flex max-h-dvh w-full flex-col overflow-hidden rounded-lg bg-white outline">
            <div class="flex min-h-11 place-items-center bg-gray-200 px-4 font-bold">
              <p>リスト追加</p>
              <button
                class="absolute right-0 z-10 flex size-11 items-center justify-center text-gray-800 hover:text-gray-600"
                popovertarget="editListPopover"
                popovertargetaction="hide"
              >
                <i class="i-mdi-close size-6"></i>
              </button>
            </div>

            <div class="flex-1 overflow-auto p-4">
              <div class="flex flex-col">
                <p class="text-base font-bold">
                  YouTubeの動画URLを入力してください（改行で複数入力可）
                </p>

                <form @submit="onUrlsSubmit" class="flex flex-col gap-2">
                  <textarea v-model="urlInputTextarea" class="bg-gray-200 p-2" rows="5"></textarea>
                  <button
                    type="submit"
                    class="rounded-md bg-blue-600 px-4 py-2 text-white hover:bg-blue-700"
                  >
                    追加
                  </button>
                </form>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
