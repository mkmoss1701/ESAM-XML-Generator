<script setup>
import { ref, computed, reactive } from "vue";
import XmlViewer from "vue3-xml-viewer";
import SegmentTypeId from "./components/SegmentTypeId.vue";
import { getSegmentPair } from "./modules/SegmentPairs";


const aquisitionPointIdentity = ref('');

//Init the array we will use for segment state
const segmentSource = [];

//Create a reactive proxy to manage state for the segmet array.
const segments = reactive(segmentSource);

/**
 * Call addSegment to push in a default blank value to get the output rolling.
 */
addSegment();

/**
 * Using a computed property to consistently rebuild the XML output as
 * the segments object and other options update.
 *
 * This looks a bit messy, but for now we need to concat bit of the XML into a single
 * stirng to feed it into the viewer component.
 */
const signalXML = computed(() => {
  const xmlHead = `<SignalProcessingNotification xmlns="urn:cablelabs:iptvservices:esam:xsd:signal:1" xmlns:sig="urn:cablelabs:md:xsd:signaling:3.0" xmlns:common="urn:cablelabs:iptvservices:esam:xsd:common:1" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" acquisitionPointIdentity="ExampleESAM">

    <common:BatchInfo batchId="1">
      <common:Source xsi:type="content:MovieType" />
    </common:BatchInfo>`;

  const xmlEnd = `</SignalProcessingNotification>`;
  let segmentsOutput = ``;

  segments.forEach((segment, index) => {
    segmentsOutput = segmentsOutput.concat(getSegmentPair(segment, index, aquisitionPointIdentity.value));
  });

  return `${xmlHead}${segmentsOutput}${xmlEnd}`;
});

function addSegment() {
  segments.push({ segmentTypeId: 52, segmentStart: "", segmentDuration: "" });
}

function removeSegment(index) {
  segments.splice(index, 1);
}

function copyOutput() {
 navigator.clipboard.writeText(signalXML.value);
}

</script>

<template>
  <main style="display: grid; grid-template-columns: 1fr 1fr; gap: 2em">
    <section class="segment-form">
      <h1>SCTE-35 Marker XML Generator</h1>
      <div class="segment-group">
        <label>Aquisition Point Identity</label>
        <input type="text" v-model="aquisitionPointIdentity"/>
      </div>

      <div v-for="(segment, index) in segments" class="segment-input-container">
        <h2>Segment {{ index + 1 }}</h2>
        <div class="segment-group">
          <label>Type ID:</label>
          <segmentTypeId v-model="segments[index].segmentTypeId" />
        </div>
        <div class="segment-group">
          <label>Segment Start:</label>
          <input type="number" name="segmentStart" v-model="segments[index].segmentStart" :data-valid="segments[index].segmentStart != ''"/>
        </div>
        <div class="segment-group">
          <label>Segment Duration:</label>
          <input type="number" name="segmentDuration" v-model="segments[index].segmentDuration" :data-valid="segments[index].segmentDuration != ''"/>
        </div>
        <button @click.prevent="removeSegment(index)" class="button-delete">Delete</button>
      </div>

      <div style="display:flex; gap:1rem;">
        <button @click="addSegment" class="button-add">Add Segment</button>
        <button @click="copyOutput">Copy Output</button>
      </div>
      
    </section>

    <section class="xml-preview">
      <XmlViewer :xml="signalXML" />
    </section>
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
