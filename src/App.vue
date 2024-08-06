<script setup>
import { ref, computed, reactive } from 'vue';
import XmlViewer from 'vue3-xml-viewer';

const segmentSource = [];
const segments = reactive(segmentSource);
addSegment();

const signalXML = computed(() => {
  const xmlHead = `<SignalProcessingNotification xmlns="urn:cablelabs:iptvservices:esam:xsd:signal:1" xmlns:sig="urn:cablelabs:md:xsd:signaling:3.0" xmlns:common="urn:cablelabs:iptvservices:esam:xsd:common:1" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" acquisitionPointIdentity="ExampleESAM">

    <common:BatchInfo batchId="1">
      <common:Source xsi:type="content:MovieType" />
    </common:BatchInfo>`;

  const xmlEnd = `</SignalProcessingNotification>`;
  let segmentsOutput = ``;
  
  segments.forEach((segment, index) => {
    segmentsOutput = segmentsOutput.concat(
    `<ResponseSignal acquisitionPointIdentity="ExampleESAM" acquisitionSignalID="1" signalPointID="${segment.segmentStart}" action="create">
      <sig:NPTPoint nptPoint="${segment.segmentStart}"/>
      <sig:SCTE35PointDescriptor spliceCommandType="06">
        <sig:SegmentationDescriptorInfo segmentEventId="1" segmentTypeId="52" upidType="9" upid="1" duration="PT${segment.segmentDuration}S" segmentNumber="${index}" segmentsExpected="1"/>
      </sig:SCTE35PointDescriptor>
    </ResponseSignal>
    <ConditioningInfo acquisitionSignalIDRef="1" startOffset="PT60S" duration="PT${segment.segmentDuration}S"/>`);
  }); 

return `${xmlHead}${segmentsOutput}${xmlEnd}`;
});

function addSegment() {
  segments.push({segmentStart:'', segmentDuration:''});
}

function removeSegment(index) {
  segments.splice(index, 1);
}
</script>

<template>
  <main style="display:grid; grid-template-columns: 1fr 1fr; gap:2em;">
    <section>
        <div v-for="(segment, index) in segments">
          <pre>{{segments.value}}</pre>
            <div>
                <label>Segment Start {{ index + 1 }}</label>
                <input type="number" name="segmentStart" v-model="segments[index].segmentStart"/>
            </div>
            <div>
                <label>Segment Duration {{ index +1 }}</label>
                <input type="number" name="segmentDuration" v-model="segments[index].segmentDuration"/>
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
