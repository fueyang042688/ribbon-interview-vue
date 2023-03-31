<template>
  <v-app>
    <v-app-bar>
      <v-container class="d-flex align-center py-0">
        <v-app-bar-title class="pl-0">
          <div class="d-flex align-center">Ribbon</div>
        </v-app-bar-title>
      </v-container>
    </v-app-bar>

    <v-main>
      <section id="hero">
        <v-sheet class="d-flex align-center pb-16" color="grey-darken-3">
          <v-container class="text-center">
            <v-responsive class="mx-auto">
              <h3 class="text-h3">Try Ribbon's all new features</h3>

              <p class="mt-4 text-medium-emphasis">
                Our all-in-one platform gives you the banking, accounting,
                fundraising, and organizational tools you need to build a
                successful charity under the umbrella of your fiscal sponsor.
              </p>
            </v-responsive>
          </v-container>
        </v-sheet>
      </section>

      <v-sheet>
        <section id="filter">
          <v-container>
            <v-row justify="space-between">
              <v-col cols="auto">
                <h2 class="text-h4">Ribbon Donor List</h2>

                <p class="text-primary mt-3">In Beta now!</p>

                <p class="mt-3">See all those that have given in one place!</p>
              </v-col>
            </v-row>
            <v-row>
              <v-col>
                <v-data-table
                  v-if="donors"
                  :headers="headers"
                  :items="donors.data"
                  :search="search"
                  :loading="loading"
                  :options="pagination"
                  :server-items-length="donors.total"
                  :footer-props="{
                    'items-per-page-options': [5, 10, 20],
                  }"
                  id="donor-table"
                  class="elevation-1"
                >
                  <template v-slot:progress v-if="loading">
                    <v-progress-linear
                      indeterminate
                      color="#00754A"
                    ></v-progress-linear>
                  </template>
                  <template v-slot:top>
                    <v-text-field
                      v-model="search"
                      label="Search"
                      class="mx-4"
                    ></v-text-field>
                  </template>
                  <template #header="{ headers, disableSort }">
                    <tr>
                      <th
                        v-for="header in headers"
                        :key="header.text"
                        :class="[
                          'text-left',
                          'table-header',
                          { 'tet-disabled': disableSort(header) },
                        ]"
                        @click="sortBy(header.value)"
                      >
                        <v-icon small>arrow-upward</v-icon>
                        {{ header.text }}
                      </th>
                    </tr>
                  </template>
                  <template #item="{ item }">
                    <tr :key="item.id" class="donor-table-row">
                      <td>{{ item.full_name }}</td>
                      <td>{{ item.email }}</td>
                      <td>{{ formatCurrency(item.total_donations) }}</td>
                      <td>{{ formatDate(item.first_donation) }}</td>
                    </tr>
                  </template>
                </v-data-table>
              </v-col>
            </v-row>
          </v-container>
        </section>
      </v-sheet>

      <v-sheet class="py-16" color="#1818181a">
        <section id="grid">
          <v-container>
            <v-row justify="space-between">
              <v-col cols="auto">
                <v-responsive width="350">
                  <h2 class="text-h4">Show your support feature</h2>
                  <p class="text-success mt-3">Available now!</p>
                  <p class="mt-3">
                    Easily send messages to those that have given!
                  </p>
                </v-responsive>
              </v-col>
              <v-sheet width="400" class="mx-auto">
                <v-form
                  v-model="valid"
                  validate-on="submit"
                  @submit.prevent="submit"
                >
                  <v-textarea
                    v-model="message"
                    :rules="messageRules"
                    label="Message"
                  ></v-textarea>
                  <v-text-field
                    v-model="email"
                    :rules="emailRules"
                    label="Email"
                  ></v-text-field>
                  <v-text-field
                    v-model="donor_id"
                    label="Donor Id"
                  ></v-text-field>
                  <v-btn type="submit" block class="mt-2">Send</v-btn>
                </v-form>
              </v-sheet>
            </v-row>
          </v-container>
        </section>
      </v-sheet>
    </v-main>

    <v-footer>
      <v-container
        class="text-overline d-flex align-center justify-space-between"
      >
        <div>Copyright &copy; 2023 Flourish Change Inc dba Ribbon</div>

        <v-icon icon="mdi-bank" size="x-large" />
      </v-container>
    </v-footer>
  </v-app>
</template>

<script>
  import axios from 'axios';
  export default {
    name: 'App',

    data() {
      return {
        donors: null,
        valid: false,
        email: '',
        donor_id: '',
        message: '',
        emailRules: [
          (value) => {
            if (value) return true;

            return 'E-mail is required.';
          },
        ],
        messageRules: [
          (value) => {
            if (value) return true;

            return 'Message is required.';
          },
        ],
        headers: [
          { text: 'Name', value: 'full_name' },
          { text: 'Email', value: 'email' },
          { text: 'Total Donations', value: 'total_donations' },
          { text: 'First Donation', value: 'first_donation' },
        ],
        pagination: {
          descending: false,
          rowsPerPage: 10,
          page: 1,
        },
        search: '',
        loading: false,
      };
    },
    mounted() {
      this.fetchDonors();
    },
    methods: {
      async fetchDonors() {
        try {
          this.loading = true;
          const response = await axios.get(
            'https://interview.ribbon.giving/api/donors'
          );
          this.donors = response.data;
          this.loading = false;
        } catch (error) {
          console.error(error);
        }
      },
      searchDonors(item, search) {
        return (
          item.full_name.toLowerCase().includes(search.toLowerCase()) ||
          item.email.toLowerCase().includes(search.toLowerCase())
        );
      },
      formatDate(date) {
        const options = { year: 'numeric', month: 'short', day: 'numeric' };
        return new Date(date).toLocaleDateString(undefined, options);
      },
      formatCurrency(amount) {
        const formatter = new Intl.NumberFormat('en-US', {
          style: 'currency',
          currency: 'USD',
          minimumFractionDigits: 2,
        });
        return formatter.format(amount);
      },
      async submit() {
        // Send message to server.
      },
    },
    computed: {
      filterDonors() {
        if (!this.search) {
          return this.donors.data;
        }

        return this.donors.data.filter((item) =>
          this.searchDonors(item, this.search)
        );
      },
    },
  };
</script>
<style lang="scss">
  @import './styles/app.scss';
</style>
