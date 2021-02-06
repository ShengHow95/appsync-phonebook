<template>
  <v-container>
    <v-data-table
      :headers="headers"
      :items="contactList"
      :items-per-page="25"
      class="elevation-1"
    >
      <template #top>
        <v-toolbar flat>
          <v-toolbar-title>Contact Information</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title>
                <span class="headline">Delete Contact</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="deleteItem.name"
                        label="Name"
                        disabled
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="deleteItem.contact"
                        label="Contact"
                        disabled
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="deleteItem.organization"
                        label="Organization"
                        disabled
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="cancelDelete">
                  Cancel
                </v-btn>
                <v-btn color="red darken-1" text @click="confirmDelete">
                  Confirm
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogAddOrEdit" max-width="500px">
            <template #activator="{ on, attrs }">
              <v-btn color="primary" dark v-bind="attrs" v-on="on">
                New Contact
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="headline">Contact Details</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="editedItem.name"
                        label="Name"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="editedItem.contact"
                        label="Contact"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="editedItem.organization"
                        label="Organization"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="close">
                  Cancel
                </v-btn>
                <v-btn color="blue darken-1" text @click="save"> Save </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template #[`item.action`]="{ item }">
        <v-btn fab small text @click="editContact(item)">
          <v-icon small>mdi-pencil</v-icon>
        </v-btn>
        <v-btn fab small text @click="deleteContact(item)">
          <v-icon small>mdi-delete</v-icon>
        </v-btn>
      </template>
    </v-data-table>
  </v-container>
</template>

<script>
import { API, graphqlOperation } from '@aws-amplify/api'
import { listContacts } from '~/graphql/queries'
import {
  createContact,
  updateContact,
  deleteContact,
} from '~/graphql/mutations'
import {
  onCreateContact,
  onUpdateContact,
  onDeleteContact,
} from '~/graphql/subscriptions'

export default {
  components: {},
  data: () => ({
    loading: false,
    contactList: [],
    headers: [
      { text: 'Name', value: 'name', sortable: true },
      { text: 'Contact', value: 'contact', sortable: false },
      { text: 'Organization', value: 'organization', sortable: true },
      { text: '', value: 'action' },
    ],
    search: '',
    dialogAddOrEdit: false,
    dialogDelete: false,
    editedItem: {
      id: null,
      name: null,
      contact: null,
      organization: null,
    },
    deleteItem: {
      id: null,
      name: null,
      contact: null,
      organization: null,
    },
    createSubscription: null,
    updateSubscription: null,
    deleteSubscription: null,
  }),
  async mounted() {
    const response = await API.graphql(graphqlOperation(listContacts))
    this.contactList = response.data.listContacts.items

    this.createSubscription = API.graphql(
      graphqlOperation(onCreateContact)
    ).subscribe({
      next: ({ provider, value }) => {
        this.contactList.push(value.data.onCreateContact)
      },
    })

    this.updateSubscription = API.graphql(
      graphqlOperation(onUpdateContact)
    ).subscribe({
      next: ({ provider, value }) => {
        const updateIndex = this.contactList.findIndex(
          (obj) => obj.id === value.data.onUpdateContact.id
        )
        this.contactList.splice(updateIndex, 1, value.data.onUpdateContact)
      },
    })

    this.deleteSubscription = API.graphql(
      graphqlOperation(onDeleteContact)
    ).subscribe({
      next: ({ provider, value }) => {
        const deleteIndex = this.contactList.findIndex(
          (obj) => obj.id === value.data.onDeleteContact.id
        )
        this.contactList.splice(deleteIndex, 1)
      },
    })
  },
  methods: {
    close() {
      this.dialogAddOrEdit = false
      this.editedItem = {
        name: null,
        contact: null,
        organization: null,
      }
    },
    cancelDelete() {
      this.dialogDelete = false
      this.deleteItem = {
        id: null,
        name: null,
        contact: null,
        organization: null,
      }
    },
    editContact(item) {
      this.editedItem.id = item.id
      this.editedItem.name = item.name
      this.editedItem.contact = item.contact
      this.editedItem.organization = item.organization
      this.dialogAddOrEdit = true
    },
    deleteContact(item) {
      this.deleteItem.id = item.id
      this.deleteItem.name = item.name
      this.deleteItem.contact = item.contact
      this.deleteItem.organization = item.organization
      this.dialogDelete = true
    },
    async save() {
      const data = {
        id: this.editedItem.id,
        name: this.editedItem.name,
        contact: this.editedItem.contact,
        organization: this.editedItem.organization,
      }

      if (data.id) {
        this.close()
        await this.updateContact(data)
      } else {
        this.close()
        delete data.id
        await this.createContact(data)
      }
    },
    async createContact(data) {
      await API.graphql(graphqlOperation(createContact, { input: data }))
    },
    async updateContact(data) {
      await API.graphql(graphqlOperation(updateContact, { input: data }))
    },
    async confirmDelete() {
      const data = {
        id: this.deleteItem.id,
      }
      await API.graphql(graphqlOperation(deleteContact, { input: data }))
      this.dialogDelete = false
    },
  },
}
</script>
