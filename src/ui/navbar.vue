<template>
  <s-appbar id="navbar">
    <s-icon-button slot="navigation" @click="emit('toggleDrawer')">
      <s-icon name="menu"></s-icon>
    </s-icon-button>
    <div slot="headline">
      <span class="nav-title">
        Function-Plot-GUI
        <span class="version"> v{{ version }} </span>
      </span>
    </div>

    <s-tooltip slot="action" id="themeTrigger">
      <s-icon-button slot="trigger" @click="theme.toogle">
        <SIconAutoTheme v-if="theme.index === 0" />
        <s-icon
          v-else
          :name="theme.index - 1 ? 'light_mode' : 'dark_mode'"
        ></s-icon>
      </s-icon-button>
      {{ t("nav.toogleTheme") }}
    </s-tooltip>

    <s-picker
      slot="action"
      v-model.lazy="locale"
      id="language"
      :label="t('nav.language')"
    >
      <s-tooltip slot="trigger">
        <s-icon-button slot="trigger">
          <SIconLanguage />
        </s-icon-button>
        {{ t("nav.language") }}
      </s-tooltip>

      <s-picker-item value="zh-CN" selected>简体中文</s-picker-item>
      <s-picker-item value="en-US">English</s-picker-item>
    </s-picker>
    <s-tooltip slot="action">
      <s-icon-button slot="trigger" @click="openGithub">
        <SIconGtihub />
      </s-icon-button>
      {{ t("nav.repo") }}
    </s-tooltip>
  </s-appbar>
</template>

<script setup lang="ts">
import { watch } from "vue";
import { useI18n } from "vue-i18n";
import { I18nSchema } from "@/i18n";
const { locale, t } = useI18n<{ message: I18nSchema }>();

import { version } from "@/../package.json";

watch(locale, () => {
  document.documentElement.setAttribute("lang", locale.value);
  localStorage.setItem("lang", locale.value);
});

import SIconLanguage from "@/ui/icons/language.vue";
import SIconGtihub from "@/ui/icons/github.vue";
import SIconAutoTheme from "@/ui/icons/autotheme.vue";

const openGithub = () =>
  window.open("https://github.com/Linho1219/function-plot-gui");

const emit = defineEmits(["toggleDrawer"]);

import { useTheme } from "@/states";
const theme = useTheme();
</script>

<style>
#navbar {
  box-shadow: #0005 0 0 15px;
  z-index: 9;
}

#themeTrigger s-icon {
  z-index: -1;
}

.nav-title {
  font-weight: bold;
  color: var(--s-color-on-background);
}
.version {
  opacity: 0.5;
  font-size: 0.8em;
}
</style>
