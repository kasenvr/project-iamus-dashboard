<!--
//  DomainList.vue
//
//  Created by Kalila L. on 5 Sep 2020.
//  Copyright 2020 Vircadia contributors.
//
//  Distributed under the Apache License, Version 2.0.
//  See the accompanying file LICENSE or http://www.apache.org/licenses/LICENSE-2.0.html
-->
<script>
// This is temporary for rapid iteration.
/* eslint-disable */
</script>

<template>
    <v-data-table
        :headers="headers"
        :items="domains"
        sort-by="user"
        class="elevation-1"
        @click:row="rowClicked"
    >
        <template v-slot:top>
            <v-toolbar flat>
                <v-toolbar-title>Domains</v-toolbar-title>
                <v-divider
                    class="mx-4"
                    inset
                    vertical
                ></v-divider>
                <v-spacer></v-spacer>
                <!-- DOMAIN DATA DIALOG -->
                <v-dialog
                    v-model="domainDialogShow"
                    width="500"
                    color="primary"
                >
                    <v-card>
                        <v-card-title>
                            {{ domainDialog.placeName }}
                        </v-card-title>
                
                        <v-card-text class="text-left">
                            <v-list class="transparent">
                                <v-list-item>
                                    <v-list-item-content>
                                        <v-list-item-title>
                                            Place Name
                                        </v-list-item-title>
                                        <v-list-item-subtitle>
                                            {{ domainDialog.placeName }}
                                        </v-list-item-subtitle>
                                    </v-list-item-content>
                                </v-list-item>
                                <v-list-item>
                                    <v-list-item-content>
                                        <v-list-item-title>
                                            Users
                                        </v-list-item-title>
                                        <v-list-item-subtitle>
                                            {{ domainDialog.users }}
                                        </v-list-item-subtitle>
                                    </v-list-item-content>
                                </v-list-item>
                                <v-list-item>
                                    <v-list-item-content>
                                        <v-list-item-title>
                                            Ice Server
                                        </v-list-item-title>
                                        <v-list-item-subtitle>
                                            {{ domainDialog.iceServer }}
                                        </v-list-item-subtitle>
                                    </v-list-item-content>
                                </v-list-item>
                                <v-list-item>
                                    <v-list-item-content>
                                        <v-list-item-title>
                                            Domain ID
                                        </v-list-item-title>
                                        <v-list-item-subtitle>
                                            {{ domainDialog.domainID }}
                                        </v-list-item-subtitle>
                                    </v-list-item-content>
                                </v-list-item>
                                <v-list-item>
                                    <v-list-item-content>
                                        <v-list-item-title>
                                            Owner Account ID
                                        </v-list-item-title>
                                        <v-list-item-subtitle>
                                            {{ domainDialog.sponsorAccountId }}
                                        </v-list-item-subtitle>
                                    </v-list-item-content>
                                </v-list-item>
                                <v-list-item>
                                    <v-list-item-content>
                                        <v-list-item-title>
                                            Protocol
                                        </v-list-item-title>
                                        <v-list-item-subtitle>
                                            {{ domainDialog.protocol }}
                                        </v-list-item-subtitle>
                                    </v-list-item-content>
                                </v-list-item>
                                <v-list-item>
                                    <v-list-item-content>
                                        <v-list-item-title>
                                            Version
                                        </v-list-item-title>
                                        <v-list-item-subtitle>
                                            {{ domainDialog.version }}
                                        </v-list-item-subtitle>
                                    </v-list-item-content>
                                </v-list-item>
                                <v-list-item>
                                    <v-expansion-panels>
                                        <v-expansion-panel>
                                            <v-expansion-panel-header disable-icon-rotate>
                                                Public Key
                                                <template v-slot:actions>
                                                    <v-icon color="primary">mdi-information-variant</v-icon>
                                                </template>
                                            </v-expansion-panel-header>
                                            <v-expansion-panel-content>
                                                {{ domainDialog.publicKey }}
                                            </v-expansion-panel-content>
                                        </v-expansion-panel>
                                    </v-expansion-panels>
                                </v-list-item>
                            </v-list>
                        </v-card-text>
                    </v-card>
                </v-dialog>
            </v-toolbar>
        </template>
        <template v-slot:item.actions="{ item }">
            <v-icon
                small
                v-on:click.stop="deleteDomain(item.domainID, item.placeName)"
                :disabled="!canEditDomain(item.sponsorAccountId)"
            >
                mdi-nuke
            </v-icon>
        </template>
        <template v-slot:item.images="{ item }">
            <v-avatar>
                <img
                    :src="item.images.thumbnail"
                >
            </v-avatar>
        </template>
        <!-- <template v-slot:no-data>
            <v-btn color="primary" @click="initialize">Reset</v-btn>
        </template> -->
        <!-- EDIT DIALOGS -->
        <template v-slot:item.placeName="{ item }">
            <v-edit-dialog
                @save="savePlaceName(item.placeName)"
                @cancel=""
                @open="beginEditingDomain(item.domainID)"
                @close=""
                :return-value.sync="item.placeName"
            > {{ item.placeName }}
                <template v-slot:input>
                    <v-text-field
                        v-model="item.placeName"
                        label="Edit"
                        single-line
                        counter
                        :disabled="!$store.state.account.useAsAdmin"
                    ></v-text-field>
                </template>
            </v-edit-dialog>
        </template>
    </v-data-table>
</template>

<script>
import { EventBus } from '../plugins/eventBus.js';
var vue_this;
var store;
var metaverseServer;

export default {
    name: 'DomainList',
    props: {
        source: String
    },
    data: () => ({
        dialog: false,
        headers: [
            {
                text: 'Place Name',
                align: 'start',
                sortable: true,
                value: 'placeName',
            },
            { text: 'Protocol', value: 'protocol' },
            { text: 'Version', value: 'version' },
            { text: 'Users', value: 'users' },
            { text: 'Actions', value: 'actions', sortable: false },
        ],
        domainDialogShow: false,
        domainDialog: {
            placeName: '',
            users: '',
            domainID: '',
            version: '',
            protocol: '',
            publicKey: '',
            iceServer: '',
            sponsorAccountId: '',
            networkingMode: ''
        },
        // Editing Domain
        editingDomain: null,
        // Domain List
        domains: [],
    }),
    
    computed: {
    },

    watch: {
    },

    created () {
        vue_this = this;
        store = this.$store.state;
        metaverseServer = store.metaverseConfig.server;
        
        this.initialize();
    },
    
    computed: {
    },

    methods: {
        initialize () {
            this.retrieveDomainList(metaverseServer);
        },

        retrieveDomainList: function (metaverseURL) {
            var parameters = window.$.param({
                "asAdmin": store.account.useAsAdmin
            });
            parameters = "?" + parameters;

            window.$.ajax({
                type: 'GET',
                url: metaverseURL + '/api/v1/domains' + parameters,
                contentType: 'application/json',
            })
                .done(function (result) {
                    vue_this.domains = [];
                    result.data.domains.forEach(function(item, index) {
                        vue_this.domains.push(
                            {
                                placeName: item.name,
                                domainID: item.domainId,
                                users: item.total_users,
                                protocol: item.protocol_version,
                                publicKey: item.public_key,
                                version: item.version,
                                iceServer: item.ice_server_address,
                                sponsorAccountId: item.sponsor_accountid,
                                networkingMode: item.networking_mode
                            }
                        );
                    });
                })
                .fail(function (result) {
                    console.info('Failed to retrieve domain list: ', result);
                })
        },
        
        rowClicked (rowData) {
            this.domainDialogShow = true;
            this.domainDialog.placeName = rowData.placeName;
            this.domainDialog.domainID = rowData.domainID;
            this.domainDialog.version = rowData.version;
            this.domainDialog.protocol = rowData.protocol;
            this.domainDialog.publicKey = rowData.publicKey;
            this.domainDialog.iceServer = rowData.iceServer;
            this.domainDialog.sponsorAccountId = rowData.sponsorAccountId;
            this.domainDialog.networkingMode = rowData.networkingMode;
            // this.domainDialog.restricted = rowData.restricted;
            // this.domainDialog.capacity = rowData.capacity;
            // this.domainDialog.description = rowData.description;
            // this.domainDialog.maturity = rowData.maturity;
            // this.domainDialog.restriction = rowData.restriction;
            // this.domainDialog.hosts = rowData.hosts;
            // this.domainDialog.tags = rowData.tags;
            this.domainDialog.users = rowData.users;
            // this.domainDialog.numUsers = rowData.numUsers;
            // this.domainDialog.anonUsers = rowData.anonUsers;
            // this.domainDialog.meta = rowData.meta;
        },
        
        canEditDomain: function (domainOwningID) {
            return store.account.useAsAdmin || store.account.accountId === domainOwningID;
        },
        
        // BEGIN Inline Editing Functionality
        beginEditingDomain (domainID) {
            this.editingDomain = domainID;
        },
        
        savePlaceName (newPlaceName) {
            this.postUpdateDomain(this.editingDomain, {
                'place_name': newPlaceName,
            })
        },
        
        deleteDomain (domainID, placeName) {
            confirm('Are you sure you want to delete ' + placeName + '?') && this.postDeleteDomain(domainID);
        },
        // END Inline Editing Functionality
        
        // BEGIN Handling requests to the API
        postUpdateDomain (domainID, dataToUpdate) {
            var parameters = window.$.param({
                "asAdmin": store.account.useAsAdmin
            });
            parameters = "?" + parameters;

            window.$.ajax({
                type: 'POST',
                url: metaverseServer + 'api/v1/domains/' + domainID + parameters,
                contentType: 'application/json',
                data: { 
                    'domain': {
                        dataToUpdate
                    }
                }
            })
                .done(function (result) {
                    console.info('Successfully updated domain:', domainID);
                    vue_this.retrieveDomainList(metaverseServer);
                })
                .fail(function (result) {
                    console.info('Failed to update domain:', domainID);
                    vue_this.retrieveDomainList(metaverseServer);
                })
        },
        
        postDeleteDomain (domainID) {
            var parameters = window.$.param({
                "asAdmin": store.account.useAsAdmin
            });
            parameters = "?" + parameters;

            window.$.ajax({
                type: 'DELETE',
                url: metaverseServer + 'api/v1/domains/' + domainID + parameters,
            })
                .done(function (result) {
                    console.info('Successfully deleted domain:', domainID);
                    vue_this.retrieveDomainList(metaverseServer);
                })
                .fail(function (result) {
                    console.info('Failed to delete domain:', domainID);
                    vue_this.retrieveDomainList(metaverseServer);
                })
        }
        // END Handling requests to the API
    },
}
</script>