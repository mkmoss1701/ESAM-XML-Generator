<script setup>
import { ref, computed, reactive } from "vue";
import XmlViewer from "vue3-xml-viewer";
import SegmentTypeId from "./components/SegmentTypeId.vue";

const aquisitionPointIdentity = ref();
const segmentSource = [];
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
    segmentsOutput = segmentsOutput.concat(getSegmentPair(segment, index));
  });

  return `${xmlHead}${segmentsOutput}${xmlEnd}`;
});

/**
 * Lets break this up a little bit to make it easier to read later...
 */
function getSegmentPair(segment, index) {
  //Lets destructure out the pieces of a segment because we have some calculations to make, and its easier to read.
  const { segmentStart, segmentDuration, segmentTypeID } = segment;

  //Calculate the end of the ending segment
  const segmentEnd = segmentStart + segmentDuration;

  return `<ResponseSignal acquisitionPointIdentity="${
    aquisitionPointIdentity.value
  }" acquisitionSignalID="1" signalPointID="${segmentStart}" action="create">
      <sig:NPTPoint nptPoint="${segmentStart}"/>
      <sig:SCTE35PointDescriptor spliceCommandType="06">
        <sig:SegmentationDescriptorInfo segmentEventId="${index}" segmentTypeId="${segmentTypeId}" upidType="9" upid="1" duration="PT${segmentDuration}S" segmentNumber="${index}" segmentsExpected="1"/>
      </sig:SCTE35PointDescriptor>
    </ResponseSignal>
    <ConditioningInfo acquisitionSignalIDRef="1" startOffset="PT${segmentStart}S" duration="PT${segmentDuration}S"/>
    
    <ResponseSignal acquisitionPointIdentity="${
      aquisitionPointIdentity.value
    }" acquisitionSignalID="1" signalPointID="${segmentEnd}" action="create">
      <sig:NPTPoint nptPoint="${segmentEnd}"/>
      <sig:SCTE35PointDescriptor spliceCommandType="06">
        <sig:SegmentationDescriptorInfo segmentEventId="1" segmentTypeId="${
          parseInt(segmentTypeId) + 1
        }" upidType="9" upid="1" duration="PT${segmentDuration}S" segmentNumber="${index}" segmentsExpected="1"/>
      </sig:SCTE35PointDescriptor>
    </ResponseSignal>
    <ConditioningInfo acquisitionSignalIDRef="1" startOffset="PT60S" duration="PT${segmentDuration}S"/>`;
}

function addSegment() {
  segments.push({ segmentTypeId: 52, segmentStart: "", segmentDuration: "" });
}

function removeSegment(index) {
  segments.splice(index, 1);
}
</script>

<template>
  <main style="display: grid; grid-template-columns: 1fr 1fr; gap: 2em">
    <section>
      <div>
        <label>Aquisition Point Identity</label>
        <input type="text" v-model="aquisitionPointIdentity" />
      </div>

      <div v-for="(segment, index) in segments">
        <div>
          <label>Segment Type ID</label>
          <segmentTypeId v-model="segments[index].segmentTypeId" />
        </div>
        <div>
          <label>Segment Start {{ index + 1 }}</label>
          <input
            type="number"
            name="segmentStart"
            v-model="segments[index].segmentStart"
          />
        </div>
        <div>
          <label>Segment Duration {{ index + 1 }}</label>
          <input
            type="number"
            name="segmentDuration"
            v-model="segments[index].segmentDuration"
          />
        </div>
        <button @click.prevent="removeSegment(index)">Delete</button>
      </div>

      <button @click="addSegment">Add Segment</button>
    </section>

    <section>
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
