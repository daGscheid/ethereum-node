<template>
  <div
    class="col-start-3 col-end-4 gap-1 pt-4 pb-2 space-y-4 grid grid-flow-row auto-rows-max relative"
    @mousedown.prevent
  >
    <div
      v-for="item in getValidators"
      :key="item"
      ref="validatorRefs"
      class="h-[110px] w-[110px] relative flex justify-center py-1 items-center rounded-md shadow-md divide-x divide-gray-700 self-center justify-self-center cursor-pointer"
      :class="getDynamicClasses(item)"
      @click="displayMenu(item)"
      @mouseleave="hideMenu(item)"
      @mouseenter="mouseOver(item)"
    >
      <ClientLayout :client="item" />
      <Transition name="slide-fade">
        <GeneralMenu
          v-if="item.displayPluginMenu"
          :item="item"
          @switch-client="switchClient"
          @modify-service="modifyService"
          @delete-service="deleteService"
          @info-modal="infoModal"
        />
      </Transition>
    </div>
  </div>
</template>

<script setup>
import { useNodeManage } from "@/store/nodeManage";
import ClientLayout from "./ClientLayout.vue";

import GeneralMenu from "./GeneralMenu.vue";
import { computed, ref, watch } from "vue";

// Variables & Constants

const emit = defineEmits(["deleteService", "switchClient", "modifyService", "infoModal", "mouseOver", "mouseLeave"]);

const validatorRefs = ref([]);
const manageStore = useNodeManage();

// Computed & Watchers

const getValidators = computed(() => {
  let service;
  service = manageStore.newConfiguration
    .filter((e) => e.category == "validator")
    .sort((a, b) => {
      let fa = a.name.toLowerCase(),
        fb = b.name.toLowerCase();

      if (fa < fb) {
        return -1;
      }
      if (fa > fb) {
        return 1;
      }
      return 0;
    });
  return service;
});

const getValidatorRef = computed(() => {
  return validatorRefs.value.map((el, index) => ({
    ref: el,
    refId: getValidators.value[index]?.config?.serviceID,
  }));
});

watch(getValidatorRef, (newValue) => {
  manageStore.validatorRefList = newValue;
});

const getDynamicClasses = (item) => {
  if (item.hasOwnProperty("isRemoveProcessing") && item.isRemoveProcessing) {
    return "border bg-red-600 border-white hover:bg-red-600";
  } else {
    return "bg-[#212629] hover:bg-[#374045] border border-gray-700";
  }
};

// Methods

const displayMenu = (item) => {
  manageStore.newConfiguration.forEach((service) => {
    service.displayPluginMenu = false;
  });
  if (
    item.isNotConnectedToConsensus ||
    item.isNotConnectedToValidator ||
    item.isNotConnectedToMevboost ||
    item.isRemoveProcessing
  ) {
    return;
  } else {
    item.displayPluginMenu = true;
  }
};

const hideMenu = (item) => {
  item.displayPluginMenu = false;
  emit("mouseLeave", item);
};

const mouseOver = (item) => {
  if (!item.displayPluginMenu) {
    emit("mouseOver", item);
  }
};

const deleteService = (item) => {
  emit("deleteService", item);
};

const switchClient = (item) => {
  emit("switchClient", item);
};

const modifyService = (item) => {
  emit("modifyService", item);
};

const infoModal = (item) => {
  emit("infoModal", item);
};
</script>
<style scoped>
.slide-fade-enter-active {
  transition: all 0.3s ease-out;
}

.slide-fade-leave-active {
  transition: all 0.1s cubic-bezier(1, 0.5, 0.8, 1);
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  transform: translateX(20px);
  opacity: 0;
}
</style>