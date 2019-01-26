<template>
	<v-app id="app">
		<v-toolbar dark="dark" v-bind:color="isUpdate ? 'primary' : 'teal'">
			<v-toolbar-title class="headline text-uppercase">
				<span>Inventory</span>
			</v-toolbar-title>
			<v-spacer></v-spacer>
			<v-text-field dark="dark"
							append-icon="search"
							label="Search"
							single-line=''
							hide-details=''
							v-model="search"
							></v-text-field>
		</v-toolbar>
		
		<v-content>
			
			<!-- Insert Form -->
			<template>
				<v-form>
					<v-container>
						<v-layout>
							<v-flex xs12="xs12" md4="md4">
								<v-text-field
												v-model="tempItem.Name__c"
												label="Item Name"
												required="required"
												></v-text-field>
							</v-flex>
							
							<v-flex xs12="xs12" md4="md4">
								<v-text-field
												v-model="tempItem.Price__c"
												label="Price"
												required="required"
												></v-text-field>
							</v-flex>
							
							<v-flex xs12="xs12" md4="md4">
								<v-text-field
												v-model="tempItem.Quantity__c"
												label="Quantity"
												type="number"
												required="required"
												></v-text-field>
							</v-flex>
						</v-layout>
						
						<v-layout>
							<v-flex sm12="sm12">
								<v-textarea
											rows="1"
											auto-grow="auto-grow"
											v-model="tempItem.Description__c"
											label="Description">
								</v-textarea>
							</v-flex>
						</v-layout>
						
						<div class="text-xs-center">
							<v-btn v-on:click="clearTempItem()" outline="outline" color="error">
								<v-icon left="left" error="error">close</v-icon>
								{{ isUpdate ? 'Cancel' : 'Clear' }}
							</v-btn>
							
							<v-btn v-on:click="updateOrInsertTempItem()" dark="dark" outline="outline" v-bind:color="isUpdate ? 'primary' : 'teal'">
								<v-icon left="left" v-bind:color="isUpdate ? 'primary' : 'teal'">save</v-icon>
								{{ isUpdate ? 'Update' : 'Save' }}
							</v-btn>
						</div>
						
					</v-container>
				</v-form>
			</template>
			
			<!-- List -->
			<template>
				<v-data-table
								v-bind:headers="headers"
								v-bind:items="items"
								v-bind:search="search"
								class="elevation-1"
								>
					<template slot="items" slot-scope="props">
						<tr v-on:click="setItemToUpdate(props.item)">
							<td>{{ props.item.Name__c }}</td>
							<td>{{ props.item.Description__c }}</td>
							<td>{{ props.item.Price__c }}</td>
							<td>{{ props.item.Quantity__c }}</td>
						</tr>
					</template>
					<template slot="no-data">
						<v-alert v-bind:value="true" color="info" icon="warning">
							Add an Item above to show it here
						</v-alert>
					</template>
				</v-data-table>
			</template>
			
		</v-content>
	</v-app>
</template>

<script>
export default {
	name: 'app',
	data () {
		return {
			tempItem: { Name__c: '', Description__c: '', Price__c: '', Quantity__c: '' },
			isUpdate: false,
			search: '',
			headers: [
				{ text: 'Name', value: 'Name__c' },
				{ text: 'Description', value: 'Description__c' },
				{ text: 'Price', value: 'Price__c' },
				{ text: 'Quantity', value: 'Quantity__c' }
			],
			items: [
				{ Id: '1', Name__c: 'Item 1', Description__c: 'test with\nnew line 1', Price__c: 10.1, Quantity__c: 1 },
				{ Id: '2', Name__c: 'Item 2', Description__c: 'test with\nnew line 2', Price__c: 20.2, Quantity__c: 2 },
				{ Id: '3', Name__c: 'Item 3', Description__c: 'test with\nnew line 3', Price__c: 30.3, Quantity__c: 3 }
			]
		}
	},
	methods: {
		setItemToUpdate (item) {
			this.tempItem = item
			this.isUpdate = true
		},
		clearTempItem () {
			this.tempItem = { Name__c: '', Description__c: '', Price__c: '', Quantity__c: '' },
			this.isUpdate = false
			this.getAllItems()
		},
		getAllItems () {
			Visualforce.remoting.Manager.invokeAction(
				'{!$RemoteAction.VueInventoryController.getItens}',
				(result, event) => {
					if (event.status) {
						this.items = result
					} else if (event.type === 'exception') {
						console.log('exception')
					} else {
						console.log(event.type)
					}
				},
				{ escape: true }
			);
		},
		updateOrInsertTempItem () {
			Visualforce.remoting.Manager.invokeAction(
				'{!$RemoteAction.VueInventoryController.updateOrInsertItem}',
				JSON.stringify(this.tempItem),
				(result, event) => {
					this.clearTempItem()
					// console.log(result, event);
					// if (event.status) {
					//   console.log('event.status', event.status);
					// } else if (event.type === 'exception') {
					//   console.log('exception');
					// } else {
					//   console.log(event.type);
					// }
				},
				{ escape: true }
			);
		}
	},
	beforeMount () {
		this.getAllItems()
	}
}
</script>