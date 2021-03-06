<!--
 Copyright (c) 2020 - for information on the respective copyright owner
 see the NOTICE file and/or the repository at
 https://github.com/hyperledger-labs/organizational-agent
 
 SPDX-License-Identifier: Apache-2.0
-->

<template>
  <div>
    <div v-for="(item, index) in sortCredentials" v-bind:key="index">
      <v-row>
        <v-col cols="4">
          <span class="grey--text text--darken-2 font-weight-medium">
            <span v-if="item.type === CredentialTypes.OTHER.name">{{
              item.credentialDefinitionId | credentialTag
            }}</span>
            <span v-else>{{ item.type | credentialLabel }}</span>
          </span>
          <v-icon v-if="item.actions" small @click="deleteItem(item)"
            >mdi-delete</v-icon
          >
        </v-col>
        <v-col>
          <DocumentCredential
            v-bind:document="item"
            isReadOnly
            showOnlyContent
          ></DocumentCredential>
          <h4
            v-if="item.sentAt || item.receivedAt"
            class="grey--text text--darken-2"
          >
            Timestamp
          </h4>
          <v-row v-if="item.sentAt">
            <v-col class="pb-0">
              <v-text-field
                label="Sent at"
                :placeholder="
                  $options.filters.moment(item.sentAt, 'YYYY-MM-DD HH:mm')
                "
                disabled
                outlined
                dense
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row v-if="item.receivedAt">
            <v-col class="py-0">
              <v-text-field
                label="Received at"
                :placeholder="
                  $options.filters.moment(item.receivedAt, 'YYYY-MM-DD HH:mm')
                "
                disabled
                outlined
                dense
              ></v-text-field>
            </v-col>
          </v-row>
        </v-col>
      </v-row>
      <v-divider></v-divider>
    </div>
  </div>
</template>

<script>
import DocumentCredential from "@/components/credentials/DocumentCredential";
import { EventBus } from "../../main";
import { CredentialTypes } from "../../constants";

export default {
  props: {
    credentials: Array,
  },
  data: () => {
    return {
      selected: [],
      CredentialTypes: CredentialTypes,
      expanded: [],
    };
  },
  computed: {
    sortCredentials: function () {
      const sortKeys = { createdDate: "desc" };
      return this.credentials.sortByKeys(sortKeys);
    },
  },
  methods: {
    openPresentation(presentation) {
      if (presentation.id) {
        this.$router.push({
          path: `presentation/${presentation.id}`,
          append: true,
        });
      } else {
        EventBus.$emit(
          "error",
          "No details view available for presentations in public profile."
        );
      }
    },
  },
  components: {
    DocumentCredential,
  },
};
</script>
