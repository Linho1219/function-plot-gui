<template>
  <div id="output">
    <span id="outputTitle">{{ t("output.title") }} </span>
    <div id="outputBtns">
      <s-button type="text" @click="folded = !folded" id="outputFoldButton">
        <s-icon
          slot="start"
          :name="folded ? 'chevron_up' : 'chevron_down'"
        ></s-icon>
        {{ t(`output.${folded ? "expand" : "collapse"}`) }}
      </s-button>

      <s-tooltip align="right">
        <s-icon-button slot="trigger" @click="copyCode">
          <SIconCopy />
        </s-icon-button>
        {{ t("output.copy.tip") }}
      </s-tooltip>
    </div>
    <AnimatedFold :folded="folded">
      <s-scroll-view id="formattedCode">
        <pre ref="formattedCodePre">{{ formatted }}</pre>
      </s-scroll-view>
    </AnimatedFold>
  </div>
</template>

<script setup lang="ts">
import { useI18n } from "vue-i18n";
import { I18nSchema } from "@/i18n";
const { t } = useI18n<{ message: I18nSchema }>();

import SIconCopy from "@/ui/icons/copy.vue";

import JSON5 from "json5";
import base64 from "base-64";
import utf8 from "utf8";

import prettier from "prettier/standalone";
import prettierPluginBabel from "prettier/plugins/babel";
import prettierPluginEstree from "prettier/plugins/estree";
import { ref, watch } from "vue";
import { useProfile } from "@/states";
const profile = useProfile();

import AnimatedFold from "@/ui/animated/animatedFold.vue";

import { FunctionPlotOptions } from "function-plot";
const formatted = ref("");
watch(
  profile,
  () => {
    const resultObj: Omit<FunctionPlotOptions, "target"> = {
      data: profile.getPublicDatum(true),
      ...profile.getPublicOptions(),
    };
    if (profile.annotations.length)
      resultObj.annotations = profile.getPublicAnnotations();
    const code = JSON5.stringify(resultObj);
    location.hash =
      "code=" +
      encodeURIComponent(base64.encode(utf8.encode(code)).replace(/=+$/, ""));
    prettier
      .format(code, {
        parser: "json5",
        printWidth: 40,
        plugins: [prettierPluginBabel, prettierPluginEstree],
      })
      .then((value) => (formatted.value = value));
  },
  { immediate: true }
);

const folded = ref(true);

import { Snackbar } from "sober";

// const formattedCodePre = ref<HTMLPreElement>();
function copyCode() {
  try {
    navigator.clipboard.writeText(formatted.value);
    Snackbar.builder({
      text: t("output.copy.success"),
      type: "success",
    });
  } catch (e) {
    if (e instanceof Error) console.error(e);
    if (location.href.match(/^http:\/\//))
      Snackbar.builder({
        text:
          t("output.copy.fail") +
          ": Pages over HTTP are not allowed to use clipboard API",
        type: "error",
      });
    else
      Snackbar.builder({
        text: t("title.copyFail"),
        type: "error",
      });
  }
}
</script>

<style>
#output {
  border-top: var(--s-color-outline-variant) 1px solid;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  padding: 0;
  position: relative;
}

#outputTitle {
  font-size: 18px;
  font-weight: bold;
  padding: 8px 15px;
  line-height: 40px;
}

#outputBtns {
  position: absolute;
  bottom: 8px;
  right: 15px;
  display: flex;
}

#formattedCode {
  height: 200px;
  padding: 0 15px 15px 15px;
}

#formattedCode pre {
  user-select: text;
  cursor: text;
  margin: 0;
}

::selection {
  background: var(--s-color-primary);
  color: var(--s-color-on-primary);
}
</style>
