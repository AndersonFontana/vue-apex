<template>
	<v-app id="app">
		<v-toolbar dark="dark" v-bind:color="tab == 1 ? 'gray' : isUpdate ? 'primary' : 'teal'">
			<v-toolbar-title class="headline text-uppercase">
				<span>Inventory</span>
			</v-toolbar-title>
			<v-spacer></v-spacer>
			<v-text-field
				flat="flat"
				append-icon="search"
				label="Search"
				single-line=""
				hide-details=""
				v-model="search[tab]"
				solo-inverted="solo-inverted"
			></v-text-field>
			<v-tabs
				fixed-tabs="fixed-tabs"
				slot="extension"
				v-model="tab"
				centered="centered"
				color="transparent"
				slider-color="white"
			>
				<v-tab ripple="ripple">
					Items
				</v-tab>
				<v-tab ripple="ripple">
					Cases
				</v-tab>
			</v-tabs>
		</v-toolbar>
		
		<!-- Items Tab -->
		<v-content v-if="tab == 0">
			
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
					v-bind:headers="headersItem"
					v-bind:items="items"
					v-bind:search="search[tab]"
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
							Add an Item above to appear here
						</v-alert>
					</template>
				</v-data-table>
			</template>
			
		</v-content>
		
		<!-- Cases Tab -->
		<v-content v-if="tab == 1">
			
			<!-- Submit Case -->
			<template>
				<v-form action="https://webto.salesforce.com/servlet/servlet.WebToCase?encoding=UTF-8" method="POST">
					<input type="hidden" name="orgid" value="00D1U000000wI3z" />
					<input type="hidden" name="retURL" value="https://testvue-dev-ed--c.visualforce.com/apex/Inventory" />
					<v-container>
						<v-layout>
							<v-flex xs12="xs12" md4="md4">
								<v-text-field
									label="Contact Name"
									maxlength="80"
									name="name"
									size="20"
									type="text"
								></v-text-field>
							</v-flex>

							<v-flex xs12="xs12" md4="md4">
								<v-text-field
									label="Email"
									maxlength="80"
									name="email"
									size="20"
									type="email"
									required="required"
								></v-text-field>
							</v-flex>

							<v-flex xs12="xs12" md4="md4">
								<v-text-field
									label="Phone"
									maxlength="40"
									name="phone"
									size="20"
									type="tel"
								></v-text-field>
							</v-flex>

							<v-flex xs12="xs12" md4="md4">
								<v-text-field
									label="Subject"
									maxlength="80"
									name="subject"
									size="20"
									type="text"
								></v-text-field>
							</v-flex>
						</v-layout>

						<v-layout>
							<v-flex sm12="sm12">
								<v-textarea
									rows="1"
									auto-grow="auto-grow"
									name="description"
									label="Description">
								</v-textarea>
							</v-flex>
						</v-layout>
						
						<div class="text-xs-center">
							<v-btn type="reset" outline="outline" color="error">
								<v-icon left="left" color="error">close</v-icon>
								Clear
							</v-btn>
							
							<v-btn type="submit" outline="outline" color="gray">
								<v-icon left="left" color="gray">send</v-icon>
								Send
							</v-btn>
						</div>
						
					</v-container>
				</v-form>
			</template>
			
			<!-- List Cases -->
			<template>
				<v-data-table
					v-bind:headers="headersCase"
					v-bind:items="cases"
					v-bind:search="search[tab]"
					class="elevation-1"
					>
					<template slot="items" slot-scope="props">
						<td>{{ formatDate(props.item.LastModifiedDate) }}</td>
						<td>{{ props.item.Subject }}</td>
						<td>{{ props.item.Description }}</td>
						<td>{{ props.item.Status }}</td>
					</template>
					<template slot="no-data">
						<v-alert v-bind:value="true" color="info" icon="warning">
							Empty!
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
			search: ['', ''],
			tab: 0,
			headersItem: [
				{ text: 'Name', value: 'Name__c' },
				{ text: 'Description', value: 'Description__c' },
				{ text: 'Price', value: 'Price__c' },
				{ text: 'Quantity', value: 'Quantity__c' }
			],
			headersCase: [
				{ text: 'Last Modified', value: 'LastModifiedDate' },
				{ text: 'Subject', value: 'Subject' },
				{ text: 'Description', value: 'Description' },
				{ text: 'Status', value: 'Status' }
			],
			items: [
				// { Id: '1', Name__c: 'Item 1', Description__c: 'test with\nnew line 1', Price__c: 10.1, Quantity__c: 1 },
				// { Id: '2', Name__c: 'Item 2', Description__c: 'test with\nnew line 2', Price__c: 20.2, Quantity__c: 2 },
				// { Id: '3', Name__c: 'Item 3', Description__c: 'test with\nnew line 3', Price__c: 30.3, Quantity__c: 3 }
			],
			cases: [
				{ Id: 'Id', Subject: 'Subject', LastModifiedDate: 1548601181000, Status: 'Status', Description: 'Description' }
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
		formatDate (date) {
			return new Date(date).toUTCString()
		},
		getAllItems () {
			Visualforce.remoting.Manager.invokeAction(
				'{!$RemoteAction.VueInventoryController.getItens}',
				(result, event) => {
					if (event.status) {
						this.items = result
					} else {
						console.log(event.type)
					}
				},
				{ escape: true }
			);
		},
		getAllCases () {
			Visualforce.remoting.Manager.invokeAction(
				'{!$RemoteAction.VueInventoryController.getCases}',
				(result, event) => {
					if (event.status) {
						this.cases = result
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
		this.getAllCases()
	}
}
</script>