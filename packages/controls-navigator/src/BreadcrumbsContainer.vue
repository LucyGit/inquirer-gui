<template>
  <div>
    <div
      ref="answerBox"
      v-resize="calcIsMore"
      class="answer-box"
      :class="{ 'show-more': !showMore }"
      data-test="answerBox"
    >
      <div v-for="(item, index) in breadcrumbs" class="answer" data-test="breadcrumbs" :key="`answer-${index}`">
        <span class="NavigatorSummaryKeyClass">{{ item.label ? item.label + ":" : "" }} </span>

        <span v-if="showIconForError && isErrorType(item.type)" :class="getValueClass(item.type)">
          <v-tooltip location="top" max-width="350px">
            <template v-slot:activator="{ props }">
              <v-icon v-bind="props">mdi-close-circle-outline</v-icon>
            </template>
            <span>{{ item.value }}</span>
          </v-tooltip>
        </span>

        <span v-else :class="getValueClass(item.type)">{{ item.value }}</span>
      </div>
    </div>
    <div v-if="showMoreLess" class="more" data-test="moreLessButton" @click="showMore = !showMore">
      {{ showMore ? "More..." : "Less" }}
    </div>
  </div>
</template>

<script setup>
import { unref } from "vue";
import { ref, watch, nextTick, toRefs } from "vue";

const showMore = ref(true);
const showMoreLess = ref(false);
const answerBoxOverflowHeight = ref(Number.MAX_SAFE_INTEGER);
const answerBox = ref(null);

const props = defineProps({
  breadcrumbs: {
    type: Array,
    default: () => [],
  },
  showIconForError: {
    type: Boolean,
    default: false,
  },
});

const { breadcrumbs } = toRefs(props);

watch(
  breadcrumbs,
  (val) => {
    if (val && val.length > 0) {
      // Wait until the virtual dom is updated to calc heights
      nextTick(() => {
        calcIsMore();
      });
    }
  },
  {
    flush: "post",
  },
);

const calcIsMore = () => {
  if (unref(answerBox).clientHeight < unref(answerBox).scrollHeight - 8) {
    answerBoxOverflowHeight.value = unref(answerBox).clientHeight;
  }
  showMoreLess.value = unref(answerBoxOverflowHeight) < unref(answerBox).scrollHeight - 8;
};
const getValueClass = (type) => {
  if (type === "warning") {
    return "NavigatorSummaryWarningValueClass";
  } else {
    return "NavigatorSummaryValueClass";
  }
};
const isErrorType = (type) => {
  return type === "warning";
};
defineExpose({
  calcIsMore,
  showMoreLess,
});
</script>

<style lang="scss">
$line-height: 1.42rem; // Approximation of line height (`lh` not available)

.breadcrumbs-container {
  color: var(--vscode-foreground, #616161);
  .answer-box {
    /* 5 lines */
    max-height: calc(5 * #{$line-height});
    transition: 0.3s max-height ease-in-out;
    overflow: hidden;
    &.show-more {
      max-height: calc(100 * #{$line-height});
    }
    .answer {
      padding-top: 5px;
      padding-bottom: 5px;
      text-overflow: ellipsis;
      overflow: hidden;
      display: -webkit-box;
      -webkit-line-clamp: 3;
      -webkit-box-orient: vertical;
      line-break: anywhere;
      &:first-child {
        padding-top: 0px;
      }
    }
    .answer span:first-child {
      font-weight: 700;
      text-transform: capitalize;
    }
  }
  .more {
    color: var(--vscode-textLink-foreground, #3794ff);
    text-decoration-line: underline;
    cursor: pointer;
    padding-top: 5px;
    width: fit-content;
    &:hover {
      text-decoration-line: none;
    }
  }
}
.NavigatorSummaryValueClass {
  font-family: var(--vscode-font-family, Arial, Helvetica, sans-serif);
  margin-left: 4pt;
  font-weight: 800;
  font-size: 12px;
}
.NavigatorSummaryKeyClass {
  font-family: var(--vscode-font-family, Arial, Helvetica, sans-serif);
  font-size: 12px;
}
.NavigatorSummaryWarningValueClass {
  font-family: var(--vscode-font-family, Arial, Helvetica, sans-serif);
  margin-left: 4px;
  font-weight: 800;
  font-size: 12px;
  color: var(--vscode-notificationsErrorIcon-foreground, #f14c4c);
}
</style>
