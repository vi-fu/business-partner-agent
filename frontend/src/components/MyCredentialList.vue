<!--
 Copyright (c) 2020 - for information on the respective copyright owner
 see the NOTICE file and/or the repository at
 https://github.com/hyperledger-labs/organizational-agent
 
 SPDX-License-Identifier: Apache-2.0
-->

<template>
  <v-container>
    <v-data-table
      :hide-default-footer="data.length < 10"
      :loading="isBusy"
      v-model="selected"
      :headers="headers"
      :items="data"
      :show-select="selectable"
      single-select
      :sort-by="['createdDate']"
      :sort-desc="[false]"
      @click:row="open"
    >
      <template v-slot:[`item.type`]="{ item }">
        <div
          v-if="item.type === CredentialTypes.OTHER.name"
          class="font-weight-medium"
        >
          {{ item.credentialDefinitionId | credentialTag | capitalize }}
        </div>
        <div v-else class="font-weight-medium">
          {{ item.type | credentialLabel }}
        </div>
      </template>

      <template v-slot:[`item.createdDate`]="{ item }">
        {{ item.createdDate | moment("YYYY-MM-DD HH:mm") }}
      </template>

      <template v-slot:[`item.updatedDate`]="{ item }">
        {{ item.updatedDate | moment("YYYY-MM-DD HH:mm") }}
      </template>

      <template v-slot:[`item.issuedAt`]="{ item }">
        {{ item.issuedAt | moment("YYYY-MM-DD HH:mm") }}
      </template>

      <template v-slot:[`item.isPublic`]="{ item }">
        <v-icon v-if="item.isPublic" color="green"> mdi-eye </v-icon>
        <template v-else>
          <v-icon>mdi-eye-off</v-icon>
        </template>
      </template>

      <!-- <template v-slot:item="{ item }">
          <tr tag="tr"
            @click="open(item)"
          >
            <td class="font-weight-medium">
            <td>{{ item.updatedDate ? item.updatedDate : item.createdDate | moment("dddd, MMMM Do YYYY") }}</td>
            <td>
              <v-icon v-if="item.isPublic" color="green">mdi-eye</v-icon>
              <template v-else>
                <v-icon>mdi-eye-off</v-icon>
              </template>
            </td>
          </tr>
        </template> -->
    </v-data-table>
  </v-container>
</template>

<script>
import { CredentialTypes } from "../constants";
import { EventBus } from "../main";
export default {
  props: {
    type: String,
    headers: Array,
    selectable: {
      type: Boolean,
      default: false,
    },
  },
  created() {
    this.fetch(this.type);
  },
  data: () => {
    return {
      data: [],
      isBusy: true,
      selected: [],
      CredentialTypes: CredentialTypes,
    };
  },
  computed: {},
  methods: {
    fetch(type) {
      this.$axios
        .get(`${this.$apiBaseUrl}/wallet/${type}`)
        .then((result) => {
          console.log(result);
          if ({}.hasOwnProperty.call(result, "data")) {
            this.isBusy = false;

            if (type === "credential") {
              this.data = result.data.filter((item) => {
                return item.issuer;
              });
            } else {
              this.data = result.data;
            }

            console.log(this.data);
          }
        })
        .catch((e) => {
          this.isBusy = false;
          if (e.response.status === 404) {
            this.data = [];
          } else {
            console.error(e);
            EventBus.$emit("error", e);
          }
        });
    },
    open(doc) {
      console.log(doc);

      if (this.type === "document") {
        console.log(doc);
        this.$router.push({
          name: "Document",
          params: {
            id: doc.id,
            type: doc.type,
          },
        });
      } else {
        this.$router.push({
          name: "Credential",
          params: {
            id: doc.id,
            type: doc.type,
          },
        });
      }
    },
  },
};
</script>
