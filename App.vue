<template>
	<div id="app">
		<h1>EMP Tool</h1>

		<div class="inventory">
			<h4>Inventory</h4>
			<div v-if="buying.length != 0 || selling.length !=0">
				<div v-if="buying.length != 0">
					Buying items from the Emporium:
					<ul>
						<li v-for="item in buying" v-bind:key="item.name">x{{ item.quantity}} {{item.name}} = {{ item.quantity * getItem(item.name).sell }}</li>
					</ul>
				</div>
				<div v-if="selling.length != 0">
					Selling items to the Emporium:
					<ul>
						<li v-for="item in selling" v-bind:key="item.name">x{{ -item.quantity}} {{item.name}} = {{ -item.quantity * getItem(item.name).buy }}</li>
					</ul>
				</div>
				Total: {{ totalExchange }} emp
				<span v-show="totalExchange != 0"><br>

					{{ (0 < totalExchange) ? 'player 🠒  emporium' : 'emporium 🠒 player'}}
				</span>
				<br>

				<button @click="clear">Clear Inventory</button>
			</div>
			<div v-if="buying.length == 0 && selling.length == 0">
				You have nothing in your inventory. Buy and sell below.
			</div>
		</div>

		<div>
			<input type="" placeholder="Search Shop" v-model="searchQuery"><br>
			<input type="checkbox" name="buyable" v-model="filterBuyable">Must be buyable from the Emporium.<br>
			<input type="checkbox" name="sellable" v-model="filterSellable">Must be sellable to the Emporium.<br>
		</div>


		<div class="listing">

			<table>
				<th>
					<td @click="setSort('name')" class="button">
						{{ sortBy == 'name' ? ( ascending ? '▴' : '▾' ) : '' }}
						Name</td>
					<td @click="setSort('buy')" class="button">
						{{ sortBy == 'buy' ? ( ascending ? '▴' : '▾' ) : '' }}

						Buy from Emporium</td>
					<td @click="setSort('sell')" class="button">
						{{ sortBy == 'sell' ? ( ascending ? '▴' : '▾' ) : '' }}

						Sell to Emporium</td>

					<td data-headerinfo>Buy 64</td>
					<td data-headerinfo>Exchange</td>
					<td data-headerinfo>Sell 64</td>
				</th>
				<tr v-for="entry in filteredEntries" class="entry" v-bind:key="entry.name">
					<td class="nameColumn">
						{{entry.name}}</td>
					<td class="valueColumn">
						<span class="small">Buy Price: </span>
						{{entry.buy == -1 ? "Not bought" : entry.buy}}</td>
					<td class="valueColumn">
						<span class="small">Sell Price: </span>
						{{entry.sell == -1 ? "Not sold" : entry.sell}}
					</td>

					<td class="button" data-th @click="buy(entry.name, 64)">+64</td>

					<td v-if="inventory[entry.name] != undefined" class="quantityCell">
						<!-- {{ inventory[entry.name].quantity }} -->
						<input type="number" v-if="inventory[entry.name] != undefined" v-model="inventory[entry.name].quantity" @keypress="isNumber($event)" @input="calculate">
					</td>
					<td v-if="inventory[entry.name] == undefined" @click="buy(entry.name, 0)">0</td>

					<td class="button" @click="sell(entry.name, 64)" data>-64</td>
				</tr>

				<!-- This bottom row maintains row width when no items match		 -->
				<tr class="entry">
					<td class="nameColumn"></td>
					<td class="valueColumn"></td>
					<td class="valueColumn"></td>
					<td></td>
					<td></td>
					<td></td>
				</tr>

				<span v-show="filteredEntries.length == 0">No items match criteria!</span>
			</table>

		</div>
	</div>
</template>

<script>
// import data from './data/emp-config.json'
	var data = {}

export default {
	name: 'app',
	components: {
	},
	data () {
		return {
			listingEntries:[],
			searchQuery: "",
			sortBy: "name",
			ascending: false,
			filterBuyable: false,
			filterSellable: false,
			inventory: {},
			buying: [],
			selling: []
		}
	},
	mounted () {
		// import('./data/emp-config.json').then(({default: dat}) => {

		fetch('emp-config.json')
			.then(response => response.json())
			.then(data => {
				console.log(data);
				this.listingEntries = data.map((item) => { return { name: item.name, buy: item.sell, sell: item.buy }})
			})

		// data.forEach((item) => {
		// this.inventory[item.name] = { name: item.name, quantity: 0 }
		// })
		// this.$forceUpdate();
	},
	computed: {
		filteredEntries () {
			var filtered = this.listingEntries.filter(this.filter).sort(this.sort)
			if (this.ascending) filtered.reverse()

			return filtered
		},
		totalExchange () {
			var total = 0
			for (var itemB of this.buying)
				total += itemB.quantity * this.getItem(itemB.name).sell

			for (var itemS of this.selling)
				total += itemS.quantity * this.getItem(itemS.name).buy

			return total

		}
	},
	methods: {
		isNumber: function(evt) {
			evt = (evt) ? evt : window.event;
			var charCode = (evt.which) ? evt.which : evt.keyCode;
			if ((charCode > 31 && (charCode < 48 || charCode > 57)) && charCode !== 46) {
				evt.preventDefault();
			} else {
				this.calculate()
				return true;
			}
		},

		setSort (key) {
			if (key == this.sortBy)
				this.ascending = !this.ascending
			this.sortBy = key
		},
		cull () {
			Object.keys(this.inventory).forEach((i) => {
				if (this.inventory[i].quantity == 0 || this.inventory[i].quantity == "0" || this.inventory[i].quantity == "")
					delete this.inventory[i]
			})
		},
		buy (itemName, quantity) {
			this.cull()
			if (this.inventory[itemName] == undefined) {
				this.inventory[itemName] = { name: itemName, quantity: quantity }
			} else {
				this.inventory[itemName].quantity += quantity
			}

			this.buying = Object.values(this.inventory).filter((item) => { return item.quantity > 0 })
			this.selling = Object.values(this.inventory).filter((item) => { return item.quantity < 0 })


			this.$forceUpdate();

		},
		sell (itemName, quantity) {
			if (this.inventory[itemName] == undefined) {
				this.inventory[itemName] = { name: itemName, quantity: -quantity }
			} else {
				this.inventory[itemName].quantity -= quantity
			}

			this.buying = Object.values(this.inventory).filter((item) => { return item.quantity > 0 })
			this.selling = Object.values(this.inventory).filter((item) => { return item.quantity < 0 })

			this.cull()

			this.$forceUpdate();
		},
		calculate() {
			this.buying = Object.values(this.inventory).filter((item) => { return item.quantity > 0 })
			this.selling = Object.values(this.inventory).filter((item) => { return item.quantity < 0 })
			this.$forceUpdate()
		},
		sort (a,b) {
			if (this.sortBy == "") {
				return 0
			} else if (this.sortBy == "name") {
				return a[this.sortBy] > b[this.sortBy] ? 1 : -1
			}else {
				return a[this.sortBy] < b[this.sortBy] ? 1 : -1
			}
		},
		filter (entry) {
			if (this.searchQuery != "" && entry.name.toLowerCase().indexOf(this.searchQuery.toLowerCase()) == -1)
				return false

			if (entry.sell == -1 && this.filterSellable)
				return false
			if (entry.buy == -1 && this.filterBuyable)
				return false

			return true
		},
		clear: function () {
			this.inventory = {}
			this.buying = Object.values(this.inventory).filter((item) => { return item.quantity > 0 })
			this.selling = Object.values(this.inventory).filter((item) => { return item.quantity < 0 })

			this.$forceUpdate();
		},
		getItem: function (name) {
			for (var item of data)
				if (item.name == name)
					return item
		}
	}
}
</script>

<style>
#app {
	margin: 1em;

	font-family: 'Abel';font-size: 22px;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;

}
.listing {
	/* background: #ebeef2; */
}
.entry {
	/* width: 100%; */
	padding: .5em;
	box-sizing: border-box;
}
table {
	position: relative;
	background: #ebeef2;
	text-align: left;
	border: solid 10px #ebeef2;
}
th {
	display:table-header-group;
	text-align: center;
	position: sticky;
}
.button {
	cursor: pointer;
	background: white;
	border-radius: 3px;

	user-select: none;
	-moz-user-select: none;
	-khtml-user-select: none;
	-webkit-user-select: none;
	-o-user-select: none;

	text-align: center;
}
td {
	/* padding-left: .5em; */
	/* padding-right: .5em; */
}
.nameColumn {
	min-width: 17em;
	width: 17em;
}
.valueColumn { min-width: 10em;}
ul {
	margin: 0;
}
div {
	margin-bottom: 1em;
}
.inventory {
	max-height: 300px;
	min-height: 300px;
	overflow-y: scroll;
	border: solid 1px gray;
	padding: 1em;
}
/* Chrome, Safari, Edge, Opera */
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
	/* -webkit-appearance: none; */
	/* margin: 0; */
}

/* Firefox */
input[type=number] {
	/* -moz-appearance:textfield; */
	/* max-width: 50px; */
	height: 100%;
	display: inline-block;
	text-align: center;

	min-width: 10px;
	max-width: 100%;
	/* width: 100%; */
	box-sizing: border-box;
	margin: 0;
}
.quantityCell {
	/* padding: 0; */
	padding-left: 0;
	padding-right: 0;
	height: 1em;
	text-align: center;
	background: white;
}
.small {
	display: none;
}
table {
	width: 100%;
}
@media only screen and (max-width: 1050px) {
	thead th:not(:first-child) {
		/* display: none; */
	}

	td, th {
		display: block;
		text-align: center;
	}
	.nameColumn {
		border-bottom: 1px solid gray;
		margin-top: 1.5em;
	}

	td[data-th]:before  {
		content: attr(data-th);
	}
	td[data-headerinfo] {
		display: none;
	}
	.small {
		display: inline;
	}
	.nameColumn {
		min-width: 100%;
		width: 100%;
	}
}
</style>
