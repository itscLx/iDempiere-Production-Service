<template>
	<div>
		<div class="header">
			<header class="header">
				<div class="logo">
					<img src="./assets/logo.png" alt="Logo" />
				</div>
			</header>
		</div>
		<div class="switch-container" style="align-items: center">
			<div class="float-left" style="margin-top: 5px; margin-left: 30px">
				<h3>Production Orders</h3>
			</div>
			<div
				class="float-right switch-container"
				style="margin-top: 10px; margin-right: 30px">
				<label for="switch" class="switch-label">Include Produced:</label>
				<input
					type="checkbox"
					id="switch"
					v-model="showNotCompleted"
					class="switch-input" />
				<label
					for="switch"
					:class="{
						switch: true,
						'switch-produced': showNotCompleted,
					}"></label>
			</div>
		</div>

		<div class="product-table">
			<table class="table">
				<thead>
					<tr>
						<th>Production Order ID</th>
						<th>Production Order Name</th>
						<th>Finished Product Name</th>
						<th>Quantity</th>
						<th>Planned production date</th>
						<th>Status</th>
						<th>Action</th>
					</tr>
				</thead>
				<tbody>
					<tr
						v-for="product in filteredProducts"
						:key="product.productionOrderId">
						<td>{{ product.productionOrderId }}</td>
						<td>{{ product.productionOrderName }}</td>
						<td>{{ product.finishedProductName }}</td>
						<td>{{ product.quantity }} {{ product.quantityUnitCode }}</td>
						<td>{{ format_date(product.plannedEndDateTime, "DD.MM.YYYY") }}</td>
						<td>
							<span v-if="product.produced" class="completed-status"
								>Completed</span
							>
							<span v-else class="not-completed-status">Not Completed</span>
						</td>
						<td>
							<button
								@click="handleActionButton(product)"
								class="produce-button"
								:class="{
									'red-button': product.produced,
									'green-button': !product.produced,
								}">
								{{ product.produced ? "Reset" : "Produce" }}
							</button>
						</td>
					</tr>
				</tbody>
			</table>
			<h3>Confirmation</h3>
			<table class="table">
				<thead>
					<tr>
						<th>Production Order ID</th>
						<th>Yield quantity</th>
						<th>Actual Startdate</th>
						<th>Actual Enddate</th>
						<th>Action</th>
					</tr>
				</thead>
				<tbody>
					<tr v-for="confirmation in confirmations" :key="confirmation.orderId">
						<td>{{ confirmation.orderId }}</td>
						<td>
							{{ confirmation.yieldQuantity }}
							{{ confirmation.yieldQuantityUnitCode }}
						</td>
						<td>{{ format_date(confirmation.actualStartDateTime) }}</td>
						<td>{{ format_date(confirmation.actualEndDateTime) }}</td>
						<td>
							<button
								@click="handleResetButton(confirmation)"
								class="produce-button red-button">
								Reset
							</button>
						</td>
					</tr>
				</tbody>
			</table>
		</div>
	</div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import axios from "axios";
import moment from "moment";

interface Product {
	productionOrderId: number;
	productionOrderName: string | null;
	finishedProductId: number;
	finishedProductName: string;
	quantity: number;
	quantityUnitCode: string;
	plannedStartDateTime: string | null;
	plannedEndDateTime: string | null;
	components: Component[];
	produced: boolean;
}

interface Component {
	productId: number;
	productName: string;
	quantity: number;
	quantityUnitCode: string;
}

interface Confirmation {
	orderId: number;
	yieldQuantity: number;
	yieldQuantityUnitCode: string;
	actualStartDateTime: string;
	actualEndDateTime: string;
}

export default defineComponent({
	data() {
		return {
			products: [] as Product[],
			confirmations: [] as Confirmation[],
			showNotCompleted: false,
		};
	},
	mounted() {
		this.fetchProductionOrders();
		this.fetchConfirmations();
	},
	computed: {
		filteredProducts(): Product[] {
			if (this.showNotCompleted) {
				return this.products.filter((product) => !product.produced);
			}
			return this.products.filter((product) => product);
		},
	},
	methods: {
		async fetchProductionOrders() {
			try {
				const response = await axios.get(
					"http://172.16.7.34:8080/iDempiere/web-services/production/production-orders",
					{
						params: {
							includeProducedProductionOrders: true,
						},
					}
				);
				console.log(response.data);
				this.products = response.data as Product[];
			} catch (error) {
				console.error(error);
			}
		},
		async fetchConfirmations() {
			try {
				const response = await axios.get(
					"http://172.16.7.34:8080/iDempiere/web-services/production/confirmations"
				);
				console.log(response.data);
				this.confirmations = response.data as Confirmation[];
			} catch (error) {
				console.error(error);
			}
		},
		async handleActionButton(product: Product): Promise<void> {
			try {
				if (product.produced) {
					await axios.delete(
						`http://172.16.7.34:8080/iDempiere/web-services/production/reverse-production-confirmation/${product.productionOrderId}`
					);
					console.log("Product reset successfully!");
				} else {
					await axios.post(
						`http://172.16.7.34:8080/iDempiere/web-services/production/start-production/${product.productionOrderId}`
					);
					console.log("Product produced successfully!");
				}
			} catch (error) {
				console.error(error);
			}
		},
		async handleResetButton(confirmation: Confirmation): Promise<void> {
			try {
				await axios.delete(
					`http://172.16.7.34:8080/iDempiere/web-services/production/reverse-production-confirmation/${confirmation.orderId}`
				);
				location.reload();
			} catch (error) {
				console.error(error);
			}
		},
		format_date(value: string | null, format = "DD.MM.YYYY HH:mm:ss") {
			if (value) {
				return moment(String(value)).format(format);
			}
		},
	},
});
</script>

<style scoped>
.header {
	width: 100%;
}
.product-table {
	width: 100%;
	padding: 25px; /* Add padding to the container */
	box-sizing: border-box; /* Include padding in the total width */
}

.header {
	display: flex;
	align-items: center;
	padding: 10px;
	background-color: #f2f2f2;
}

.logo img {
	height: 40px;
}

.switch-container {
	align-items: center;
	margin: 10px 0;
}

.switch-label {
	margin-right: 10px;
}

.switch-input {
	display: none;
}

.switch {
	position: relative;
	display: inline-block;
	width: 40px;
	height: 20px;
	border-radius: 20px;
	background-color: #ccc;
	cursor: pointer;
	overflow: hidden;
}

.switch::after {
	content: "";
	position: absolute;
	top: 2px;
	left: 2px;
	width: 16px;
	height: 16px;
	border-radius: 50%;
	background-color: #4caf50; /* Green color */
	transition: transform 0.2s ease-in-out;
	transform: translateX(0);
}

.switch-input:checked + .switch::after {
	transform: translateX(100%);
	background-color: #f44336; /* Red color */
}

.table {
	width: 100%;
	border-collapse: collapse;
	margin-top: 10px;
}

.table th,
.table td {
	padding: 10px;
	text-align: left;
	border-bottom: 1px solid #ddd;
}

.table th {
	background-color: #f2f2f2;
}

.completed-status {
	color: green;
	font-weight: bold;
}

.not-completed-status {
	color: red;
	font-weight: bold;
}

.produce-button {
	padding: 5px 10px;
	background-color: #4caf50;
	border: none;
	color: white;
	cursor: pointer;
}

.produce-button:hover {
	background-color: #45a049;
}

td {
	padding-left: 20px;
	padding-right: 20px;
}

.red-button {
	color: white;
	background-color: #f44336;
}

.green-button {
	color: white;
}

.float-left {
	float: left;
}

.float-right {
	float: right;
}

.clearfix::after {
	content: "";
	display: table;
	clear: both;
}
</style>
