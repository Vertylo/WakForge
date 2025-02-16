<template>
  <tippy delay="[0, 0]" duration="0" interactive position="top" :offset="[0, -2]" :append-to="() => documentVar.body">
    <div class="item-card">
      <div class="slot-label text-center pt-1 pb-1">
        {{ item.type.validSlots[0] === 'LEFT_HAND' ? 'Ring' : $t(ITEM_SLOT_DATA[item.type.validSlots[0]].name) }} Slot
      </div>
      <div class="flex px-2 pt-1">
        <p-image :src="`https://tmktahu.github.io/WakfuAssets/items/${item.imageId}.png`" image-style="width: 40px" />
        <div class="flex flex-column ml-1">
          <div class="item-name mr-2 truncate" style="max-width: 15ch">{{ $t(`items.${item.id}`) }}</div>
          <div class="flex">
            <p-image class="mr-1" :src="`https://tmktahu.github.io/WakfuAssets/rarities/${item.rarity}.png`" image-style="width: 12px;" />
            <p-image class="mr-1" :src="`https://tmktahu.github.io/WakfuAssets/itemTypes/${item.type.id}.png`" image-style="width: 18px;" />
            <div v-if="LEVELABLE_ITEMS.includes(item.type.id)">Item Level: 50</div>
            <div v-else>Lvl: {{ item.level }}</div>
            <div v-if="item.type.validSlots[0] === ITEM_SLOT_DATA.FIRST_WEAPON.id" class="ml-1">
              {{ item.type.disabledSlots.includes(ITEM_SLOT_DATA.SECOND_WEAPON.id) ? '(2H)' : '(1H)' }}
            </div>
          </div>
        </div>
        <div class="flex-grow-1" />

        <div class="flex flex-column">
          <p-button icon="pi pi-plus" class="action-button mb-1" @click="onEquipItem(item, $event)" />

          <tippy placement="left">
            <p-button icon="pi pi-question-circle" class="action-button" @click="onGotoEncyclopedia(item)" />
            <template v-slot:content> <div class="simple-tooltip">Open Encyclopedia Page</div></template>
          </tippy>
        </div>
      </div>
    </div>

    <template v-slot:content>
      <div v-if="item" class="item-card-tooltip">
        <div class="effect-header flex pt-2 px-1">
          <p-image :src="`https://tmktahu.github.io/WakfuAssets/items/${item.imageId}.png`" image-style="width: 40px" />
          <div class="flex flex-column ml-1">
            <div class="item-name mr-2">{{ $t(`items.${item.id}`) }}</div>
            <div class="flex">
              <p-image class="mr-1" :src="`https://tmktahu.github.io/WakfuAssets/rarities/${item.rarity}.png`" image-style="width: 12px;" />
              <p-image class="mr-1" :src="`https://tmktahu.github.io/WakfuAssets/itemTypes/${item.type.id}.png`" image-style="width: 18px;" />
              <div v-if="LEVELABLE_ITEMS.includes(item.type.id)">Item Level: 50</div>
              <div v-else>Lvl: {{ item.level }}</div>
              <div v-if="item.type.validSlots[0] === ITEM_SLOT_DATA.FIRST_WEAPON.id" class="ml-1">
                {{ item.type.disabledSlots.includes(ITEM_SLOT_DATA.SECOND_WEAPON.id) ? '(2H)' : '(1H)' }}
              </div>
            </div>
          </div>
        </div>

        <ItemStatList :item="item" />
      </div>
    </template>
  </tippy>
</template>

<script setup>
import { inject } from 'vue';
import { useConfirm } from 'primevue/useconfirm';

import { LEVELABLE_ITEMS, ITEM_SLOT_DATA } from '@/models/useConstants';
import { useEncyclopedia } from '@/models/useEncyclopedia';
import { useItems } from '@/models/useItems';

import ItemStatList from '@/components/characterSheet/ItemStatList.vue';

let props = defineProps({
  item: {
    type: Object,
    default: () => {},
  },
});

let documentVar = document;
const confirm = useConfirm();

const currentCharacter = inject('currentCharacter');

const { getItemEncyclopediaUrl } = useEncyclopedia();
const { equipItem } = useItems(currentCharacter);

const onEquipItem = (item, event) => {
  equipItem(item, event, confirm);
};

const onGotoEncyclopedia = (item) => {
  let url = getItemEncyclopediaUrl(item);
  window.open(url, '_blank');
};
</script>

<style lang="scss" scoped>
.item-card {
  border: 1px solid var(--highlight-50);
  width: 230px;
  height: 85px;
  margin-right: 5px;
  margin-bottom: 5px;
  border-radius: 8px;
  background: var(--background-20);
  overflow: hidden;

  &.with-stats {
    height: 215px;
    width: 310px;
  }

  .slot-label {
    background-color: var(--primary-30);
  }
}

.action-button {
  padding: 2px;
  min-width: 20px;
  max-width: 20px;
  min-height: 20px;
  max-height: 20px;

  .p-button-icon {
    font-size: 14px;
    font-weight: 800;
  }
}
</style>
