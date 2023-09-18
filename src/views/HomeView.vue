<script setup lang="ts">
import { ref, computed, reactive } from 'vue';
import {DateTime} from "luxon";

const defaultItems = localStorage.getItem('GemItems');

let parsedItems;

try {
  parsedItems = JSON.parse(defaultItems);
} catch (e) {
  parsedItems = [];
}

interface GemItem { name: string; total: number; remaining: number; start_date: Date; end_date: Date; min_payment: number; }

const items: GemItem[] = reactive(parsedItems);

const show_add = ref(false);

const item_name = ref('');
const item_total = ref('');
const item_remaining = ref('');
const item_start_date = ref('');
const item_end_date = ref('');
const item_min_payment = ref('');

const addItem = () => {
  items.push({ name: item_name.value, total: item_total.value, remaining: item_remaining.value, start_date: item_start_date.value, end_date: item_end_date.value, min_payment: item_min_payment.value || 0 });
  saveData();
  item_name.value = '';
  item_total.value = '';
  item_remaining.value = '';
  item_start_date.value = '';
  item_end_date.value = '';
  item_min_payment.value  = '';
}

const deleteItem = (index) => {
  items.splice(index, 1);
  saveData();
}

const adjustItem = (item: GemItem) => {
  let new_remaining = prompt("Please enter new remaining", item.remaining);
  if (new_remaining != null) {
    item.remaining = new_remaining;
  }
  saveData();
}

const saveData = () => {
  console.log(JSON.stringify(items));
  localStorage.setItem('GemItems', JSON.stringify(items));
}


function calculateItemRepayment(item: GemItem): number {
  const total = item.total;
  const remaining = item.remaining;
  const startDateStr = item.start_date;
  const endDateStr = item.end_date;
  const minPayment = item.min_payment;

  const startDate = DateTime.fromISO(startDateStr);
  const endDate = DateTime.fromISO(endDateStr);
  const firstRepayment = DateTime.fromISO(DateTime.now());

  // repayment every month on the same day
  let repaymentDay = firstRepayment.day;

  // calculate how many repayments there can be until end date
  const daysToRepayLeft = Math.ceil(endDate.diff(DateTime.now()).as('days'));
  const monthsLeftToRepay = Math.ceil(endDate.diff(firstRepayment).as('months'));

  if (minPayment) {
    return minPayment;
  }

  return Math.ceil(remaining / monthsLeftToRepay * 100) / 100;
}

const totalToPay = computed(() => {
  let total = 0;

  for (const item of items) {
    const repayment = calculateItemRepayment(item)
    total+= repayment;
  }
  return total.toFixed(2);
})
</script>

<template>
  <main>
    <h3>Purchases</h3>
    <button v-if="!show_add" v-on:click="show_add = true;">Add Items</button>
    <form v-on:submit.prevent="addItem()" v-if="show_add">
    <button v-if="show_add" v-on:click="show_add = false;">Hide</button>
       <div>
        <label>Name</label>
        <div><input type="text" v-model="item_name" /></div>
       </div>
       <div>
        <label>Total</label>
        <div><input type="number" step="0.01" v-model="item_total" /></div>
       </div>
       <div>
        <label>Remaining</label>
        <div><input type="number" step="0.01" v-model="item_remaining" /></div>
       </div>
       <div>
        <label>Start Date</label>
        <div><input type="date" v-model="item_start_date" /></div>
       </div>
       <div>
        <label>End Date</label>
        <div><input type="date" v-model="item_end_date" /></div>
       </div>
       <div>
        <label>Minimum Payment</label>
        <div><input type="number" step="0.01" v-model="item_min_payment" /></div>
       </div>
       <button type="submit">Add</button>
    </form>
    <table v-if="items.length > 0">
      <tr>
        <th>Name</th>
        <th>Total</th>
        <th>Remaining</th>
        <th>Start Date</th>
        <th>End Date</th>
        <th>Minimum Payment</th>
        <th>Calculated Repay</th>
        <th></th>
        <th></th>
      </tr>
      <tr v-for="(item, index) in items">
        <td>{{ item.name }}</td>
        <td>${{ item.total }}</td>
        <td>${{ item.remaining }}</td>
        <td nowrap="nowrap">{{ item.start_date }}</td>
        <td nowrap="nowrap">{{ item.end_date }}</td>
        <td><span v-if="item.min_payment > 0">${{ item.min_payment }}</span></td>
        <td>${{ calculateItemRepayment(item).toFixed(2) }}</td>
        <td><button v-on:click="adjustItem(item)">Adjust</button></td>
        <td><button v-on:click="deleteItem(index)">X</button></td>
      </tr>
    </table>
    <h3>Calculation</h3>
    <p style="font-size: 1.3rem">
    Total to pay today: <mark>${{ totalToPay }}</mark>
    </p>
  </main>
</template>
