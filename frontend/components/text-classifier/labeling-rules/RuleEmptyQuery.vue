<!--
  - coding=utf-8
  - Copyright 2021-present, the Recognai S.L. team.
  -
  - Licensed under the Apache License, Version 2.0 (the "License");
  - you may not use this file except in compliance with the License.
  - You may obtain a copy of the License at
  -
  -     http://www.apache.org/licenses/LICENSE-2.0
  -
  - Unless required by applicable law or agreed to in writing, software
  - distributed under the License is distributed on an "AS IS" BASIS,
  - WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  - See the License for the specific language governing permissions and
  - limitations under the License.
  -->

<template>
  <div v-if="dataset" class="rule-labels-definition">
    <div class="rule__labels" v-if="labels.length">
      <classifier-annotation-button
        v-for="label in visibleLabels"
        :id="label.class"
        :key="`${label.class}`"
        :label="label"
        class="non-reactive label-button"
        :data-title="label.class"
        :value="label.class"
      >
      </classifier-annotation-button>
      <p v-if="!areAnnotationsInDataset" class="help-message">
        {{ messageNotAnnotation }}
      </p>
      <p class="help-message">Introduce a query to define a rule.</p>
    </div>
    <div v-else>
      <BaseFeedbackComponent
        :feedbackInput="inputForFeedbackComponent"
        @on-click="goToSettings"
        class="feedback-area"
      />
      <p class="help-message">
        {{ messageNotLabels }}
      </p>
    </div>
  </div>
</template>
<script>
import { mapActions } from "vuex";
import { getDatasetFromORM } from "@/models/dataset.utilities";
import { DatasetViewSettings } from "@/models/DatasetViewSettings";
import { AnnotationProgress } from "@/models/AnnotationProgress";

export default {
  props: {
    datasetId: {
      type: Array,
      required: true,
    },
    datasetTask: {
      type: String,
      required: true,
    },
    labelsFromORM: {
      type: Array,
      required: true,
    },
  },
  data: () => {
    return {
      shownLabels: DatasetViewSettings.MAX_VISIBLE_LABELS,
      inputForFeedbackComponent: {
        message: "Action needed: Create labels in the dataset settings",
        buttonLabels: [{ label: "Create labels", value: "CREATE_LABELS" }],
        feedbackType: "ERROR",
      },
      messageNotLabels:
        "To create new rules, you need al least two labels. We highly recommended starting by annotating some records with these labels.",
      messageNotAnnotation:
        "We highly recommended starting by annotating some records with these labels.",
    };
  },
  computed: {
    dataset() {
      return getDatasetFromORM(this.datasetId, this.datasetTask);
    },
    maxVisibleLabels() {
      return DatasetViewSettings.MAX_VISIBLE_LABELS;
    },
    labels() {
      return this.labelsFromORM.map((l) => ({ class: l, selected: false }));
    },
    query() {
      return this.dataset.query.text;
    },
    sortedLabels() {
      return this.labels.slice().sort((a, b) => (a.score > b.score ? -1 : 1));
    },
    filteredLabels() {
      return this.sortedLabels.filter((label) =>
        label.class.toLowerCase().match(this.searchText)
      );
    },
    visibleLabels() {
      const availableNonSelected =
        this.shownLabels < this.filteredLabels.length
          ? this.shownLabels
          : this.shownLabels;
      let nonSelected = 0;
      return this.filteredLabels.filter((l) => {
        if (nonSelected < availableNonSelected) {
          nonSelected++;
          return l;
        }
      });
    },
    annotationsProgress() {
      return AnnotationProgress.find(this.dataset.name);
    },
    areAnnotationsInDataset() {
      return !!this.annotationsProgress.validated;
    },
    datasetName() {
      return this.$route.params.dataset;
    },
    datasetWorkspace() {
      return this.$route.params.workspace;
    },
  },
  methods: {
    ...mapActions({
      changeViewMode: "entities/datasets/changeViewMode",
    }),
    expandLabels() {
      this.shownLabels = this.filteredLabels.length;
    },
    collapseLabels() {
      this.shownLabels = this.maxVisibleLabels;
    },
    goToSettings() {
      this.$router.push({
        name: "datasets-workspace-dataset-settings",
        params: {
          dataset: this.datasetName,
          workspace: this.datasetWorkspace,
        },
      });
    },
  },
};
</script>
<style lang="scss" scoped>
%item {
  min-width: 80px;
  max-width: 238px;
}
.rule-labels-definition {
  height: 100%;
  display: flex;
  flex-flow: column;
}
.feedback-interactions {
  &__button {
    margin-top: auto;
    margin-bottom: 0 !important;
    align-self: flex-start;
  }
}
.label-button {
  @extend %item;
}
.help-message {
  color: $black-37;
  max-width: 480px;
}
.label-button {
  margin: 5px;
}
.label-button :deep(.button) {
  justify-content: center;
}
.label-button:not(.active) :deep(.button) {
  background: #e0e1ff !important;
}
.rule {
  &__labels {
    margin-bottom: 1em;
    margin-left: -5px;
    margin-right: -5px;
  }
}
</style>
