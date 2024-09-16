<template>
    <div class="account-holder-container">
      <h1 class="title">Account Holder</h1>
  
      <div class="account-holder-form">
        <h2>{{ isEdit ? 'Edit Account Holder' : 'Add Account Holder' }}</h2>
        <form @submit.prevent="isEdit ? updateAccountHolder() : addAccountHolder()" class="form-container">
          <label for="accHID">Account Holder ID:</label>
          <input v-model="accountHolderForm.accHID" type="number" id="accHID" :disabled="isEdit" required class="input-field">
  
          <label for="accNUM">Account Number:</label>
          <input v-model="accountHolderForm.accNUM" type="text" id="accNUM" required class="input-field">
  
          <label for="accTypeID">Account Type ID:</label>
          <input v-model="accountHolderForm.accTypeID" type="number" id="accTypeID" required class="input-field">
  
          <label for="acc_holders_N">Account Holder Name:</label>
          <input v-model="accountHolderForm.acc_holders_N" type="text" id="acc_holders_N" required class="input-field">
  
          <label for="aC_Balance">Account Balance:</label>
          <input v-model="accountHolderForm.aC_Balance" type="number" id="aC_Balance" required class="input-field">
  
          <label for="cd">Creation Date:</label>
          <input v-model="accountHolderForm.cd" type="date" id="cd" class="input-field">
  
          <label for="updatedDate">Updated Date:</label>
          <input v-model="accountHolderForm.updatedDate" type="datetime-local" id="updatedDate" class="input-field">
  
          <label for="updatedBy">Updated By:</label>
          <input v-model="accountHolderForm.updatedBy" type="text" id="updatedBy" class="input-field">
  
          <div class="button-group">
            <button type="submit" class="btn primary">{{ isEdit ? 'Update' : 'Add' }}</button>
            <button @click="clearForm" type="button" class="btn secondary">Cancel</button>
          </div>
        </form>
      </div>
  
      <table class="account-holder-table">
        <thead>
          <tr>
            <th>ID</th>
            <th>Account Number</th>
            <th>Account Type ID</th>
            <th>Holder Name</th>
            <th>Balance</th>
            <th>Created Date</th>
            <th>Created By</th>
            <th>Updated Date</th>
            <th>Updated By</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="account in paginatedAccountHolders" :key="account.accHID">
            <td>{{ account.accHID }}</td>
            <td>
              <input v-if="editedRow === account.accHID" v-model="account.accNUM" type="text" class="input-field" />
              <span v-else>{{ account.accNUM }}</span>
            </td>
            <td>
              <input v-if="editedRow === account.accHID" v-model="account.accTypeID" type="number" class="input-field" />
              <span v-else>{{ account.accTypeID }}</span>
            </td>
            <td>
              <input v-if="editedRow === account.accHID" v-model="account.acc_holders_N" type="text" class="input-field" />
              <span v-else>{{ account.acc_holders_N }}</span>
            </td>
            <td>
              <input v-if="editedRow === account.accHID" v-model="account.aC_Balance" type="number" class="input-field" />
              <span v-else>{{ account.aC_Balance }}</span>
            </td>
            <td>{{ new Date(account.cd).toLocaleDateString() }}</td>
            <td>{{ account.createdBy }}</td>
            <td>
              <input v-if="editedRow === account.accHID" v-model="account.updatedDate" type="datetime-local" class="input-field" />
              <span v-else>{{ account.updatedDate ? new Date(account.updatedDate).toLocaleString() : 'N/A' }}</span>
            </td>
            <td>{{ account.updatedBy }}</td>
            <td>
              <button v-if="editedRow === account.accHID" @click="saveEdit(account)" class="btn save-btn">Save</button>
              <button v-else @click="editAccountHolder(account)" class="btn edit-btn">Edit</button>
              <button @click="deleteAccountHolder(account.accHID)" class="btn delete-btn">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>
  
      <div class="pagination">
        <button @click="previousPage" :disabled="currentPage <= 1" class="pagination-button">Previous</button>
        <span class="pagination-info">Page {{ currentPage }} of {{ totalPages }}</span>
        <button @click="nextPage" :disabled="currentPage >= totalPages" class="pagination-button">Next</button>
      </div>
    </div>
  </template>
  
  <script>
  import api from '../APIs/AccountHolderapi'; // Adjust the import path if needed
  
  export default {
    data() {
      return {
        accountHolders: [],
        currentPage: 1,
        pageSize: 10,
        editedRow: null,
        accountHolderForm: {
          accHID: null,
          accNUM: '',
          accTypeID: null,
          acc_holders_N: '',
          aC_Balance: null,
          cd: '',
          createdBy: 'system',
          updatedDate: '',
          updatedBy: '',
        },
        isEdit: false,
      };
    },
    computed: {
      paginatedAccountHolders() {
        const start = (this.currentPage - 1) * this.pageSize;
        const end = start + this.pageSize;
        return this.accountHolders.slice(start, end);
      },
      totalPages() {
        return Math.ceil(this.accountHolders.length / this.pageSize);
      },
    },
    methods: {
      async fetchAccountHolders() {
        try {
          const response = await api.get('/');
          this.accountHolders = response.data;
        } catch (error) {
          console.error('Failed to load account holders:', error);
        }
      },
      async addAccountHolder() {
        try {
          const accountHolderData = {
            accNUM: this.accountHolderForm.accNUM,
            accTypeID: this.accountHolderForm.accTypeID,
            acc_holders_N: this.accountHolderForm.acc_holders_N,
            aC_Balance: this.accountHolderForm.aC_Balance,
            cd: new Date().toISOString(),
            createdBy: this.accountHolderForm.createdBy,
            updatedDate: this.accountHolderForm.updatedDate || null,
            updatedBy: this.accountHolderForm.updatedBy || null,
          };
          const response = await api.post('/', accountHolderData);
          this.accountHolders.push(response.data);
          this.clearForm();
        } catch (error) {
          console.error('Failed to add account holder:', error);
        }
      },
      async updateAccountHolder() {
        try {
          await api.put(`/${this.accountHolderForm.accHID}`, this.accountHolderForm);
          const index = this.accountHolders.findIndex((acc) => acc.accHID === this.accountHolderForm.accHID);
          if (index !== -1) {
            this.accountHolders.splice(index, 1, { ...this.accountHolderForm });
          }
          this.clearForm();
        } catch (error) {
          console.error('Failed to update account holder:', error);
        }
      },
      async deleteAccountHolder(accHID) {
        try {
          await api.delete(`/${accHID}`);
          this.accountHolders = this.accountHolders.filter((account) => account.accHID !== accHID);
        } catch (error) {
          console.error('Failed to delete account holder:', error);
        }
      },
      editAccountHolder(account) {
        this.accountHolderForm = { ...account };
        this.isEdit = true;
        this.editedRow = account.accHID;
      },
      saveEdit(account) {
        this.updateAccountHolder();
        this.editedRow = null;
      },
      clearForm() {
        this.accountHolderForm = {
          accHID: null,
          accNUM: '',
          accTypeID: null,
          acc_holders_N: '',
          aC_Balance: null,
          cd: '',
          createdBy: 'system',
          updatedDate: '',
          updatedBy: '',
        };
        this.isEdit = false;
      },
      previousPage() {
        if (this.currentPage > 1) {
          this.currentPage--;
        }
      },
      nextPage() {
        if (this.currentPage < this.totalPages) {
          this.currentPage++;
        }
      },
    },
    created() {
      this.fetchAccountHolders();
    },
  };
  </script>
  
  
  
  <style scoped>
.account-holder-container {
  max-width: 1500px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.title {
  text-align: center;
  font-size: 2rem;
  margin-bottom: 20px;
}

.account-holder-form {
  margin-bottom: 30px;
}

.form-container {
  display: grid;
  grid-template-columns: 1fr 2fr;
  gap: 10px;
  margin-bottom: 20px;
}

.input-field {
  width: 100%;
  box-sizing: border-box;
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #ccc;
}

.button-group {
  grid-column: 1 / -1;
  display: flex;
  gap: 10px;
}

.btn {
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn.primary {
  background-color: #4CAF50;
  color: white;
}

.btn.secondary {
  background-color: #f44336;
  color: white;
}

.account-holder-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
  table-layout: fixed;
}

.account-holder-table th, .account-holder-table td {
  border: 1px solid #ddd;
  padding: 10px;
  text-align: center;
  word-wrap: break-word;
}

.account-holder-table th {
  background-color: #f2f2f2;
}

.account-holder-table td {
  max-width: 150px;
}

.pagination {
  text-align: center;
  margin-top: 10px;
}

.pagination-button {
  padding: 10px 15px;
  margin: 0 5px;
  border: none;
  background-color: #4CAF50;
  color: white;
  border-radius: 4px;
  cursor: pointer;
}

.pagination-info {
  margin: 0 10px;
}

.btn.edit-btn {
  background-color: #FFC107;
}

.btn.delete-btn {
  background-color: #f44336;
}

.btn.save-btn {
  background-color: #4CAF50;
  color: white;
}
</style>




import axios from 'axios';

// Create an Axios instance with the base URL and timeout
const AccountHolderApi = axios.create({
  baseURL: 'http://localhost:5113/api/accountholder', // Adjust the base URL to match your CoreWCF endpoint
  timeout: 10000,
});

// Exported functions for interacting with the AccountHolder API
export const getAllAccountHolders = () => AccountHolderApi.get('/');
export const getAccountHolderByIdAsync = (id) => AccountHolderApi.get(`/${id}`);
export const addAccountHolderAsync = (accountHolder) => AccountHolderApi.post('/', accountHolder);
export const updateAccountHolderAsync = (id, accountHolder) => AccountHolderApi.put(`/${id}`, accountHolder);
export const deleteAccountHolderAsync = (id) => AccountHolderApi.delete(`/${id}`);

export default AccountHolderApi;

