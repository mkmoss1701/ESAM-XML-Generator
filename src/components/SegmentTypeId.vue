<script setup>
const input = defineModel();
</script>

<template>
  <select v-model="input">
    <option value="0">0: Not Indicated</option>
    <option value="1">1: Content Identification</option>
    <option value="16">16: Program Start</option>
    <option value="18">18: Program Early Termination</option>
    <option value="19">19: Program Breakaway</option>
    <option value="20">20: Program Resumption</option>
    <option value="21">21: Program Runover: Planned</option>
    <option value="22">22: Program Runover: Unplanned</option>
    <option value="23">23: Program Overlap: Start</option>
    <option value="24">24: Program Blackout Override</option>
    <option value="25">25: Program Join</option>
    <option value="26">26: Program Immediate Resumption</option>
    <option value="32">32: Chapter Start</option>
    <option value="34">34: Break Start</option>
    <option value="48">48: Provider Advertisement</option>
    <option value="50">50: Distributor Advertisement</option>
    <option value="52">52: Provider Placement Opportunity</option>
    <option value="54">54: Distributor Placement Opportunity</option>
    <option value="56">56: Provider Overlay Placement Opportunity</option>
    <option value="58">58: Distributor Overlay Placement Opportunity </option>
    <option value="60">60: Provider Promo</option>
    <option value="62">62: Distributor Promo </option>
    <option value="64">64: Unscheduled Event</option>
    <option value="66">66: Alternate Content Opportunity</option>
    <option value="68">68: Provider Ad Block</option>
    <option value="70">70: Distributor Ad Block</option>
    <option value="80">80: Network Start</option>
  </select>
</template>
