<template>
    <div class="transaction-container">
    <h1 class="title">Transactions</h1>
    
    <button @click="toggleForm" class="btn primary">
      {{ showForm ? 'Hide Form' : 'Add Transaction' }}
    </button>

    <div v-if="showForm" class="transaction-form">
      <h2>{{ isEdit ? 'Edit Transaction' : 'Add Transaction' }}</h2>
      <form @submit.prevent="isEdit ? updateTransaction() : addTransaction()" class="form-container">
        <label for="transID">Transaction ID:</label>
        <input v-model="transactionForm.transID" type="number" id="transID" :disabled="isEdit" required class="input-field">
        
        <label for="accountID">Account ID:</label>
        <input v-model="transactionForm.accountID" type="number" id="accountID" required class="input-field">
        
        <label for="transTypeID">Transaction Type ID:</label>
        <input v-model="transactionForm.transTypeID" type="number" id="transTypeID" required class="input-field">
        
        <label for="amount">Amount:</label>
        <input v-model="transactionForm.amount" type="number" step="0.01" id="amount" required class="input-field">
        
        <label for="date">Date:</label>
        <input v-model="transactionForm.date" type="datetime-local" id="date" required class="input-field">
        
        <label for="transaction_type">Transaction Type:</label>
        <input v-model="transactionForm.transaction_type" type="text" id="transaction_type" required class="input-field">
        
        <label for="updatedDate">Updated Date:</label>
        <input v-model="transactionForm.updatedDate" type="datetime-local" id="updatedDate" class="input-field">
        
        <div class="button-group">
          <button type="submit" class="btn primary">{{ isEdit ? 'Update' : 'Add' }}</button>
          <button @click="clearForm" type="button" class="btn secondary">Cancel</button>
        </div>
      </form>
  
    </div>
  
      <table class="transaction-table">
        <thead>
          <tr>
            <th>Transaction ID</th>
            <th>Account ID</th>
            <th>Type ID</th>
            <th>Amount</th>
            <th>Date</th>
            <th>Transaction Type</th>
            <th>Created By</th>
            <th>Created Date</th>
            <th>Updated By</th>
            <th>Updated Date</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="transaction in paginatedTransactions" :key="transaction.transID">
            
            <td>{{ transaction.transID }}</td>
           
            <td>{{ transaction.accountID }}</td>
            <td>{{ transaction.transTypeID }}</td>
            <td> <input v-if="editedRow === transaction.transID"
            v-model="transaction.amount"
            class="input-field"
            type=number
            />
            <span v-else>{{ transaction.amount }}</span>
            </td>
       
            <td>{{ new Date(transaction.date).toLocaleString() }}</td>
            
            <td>
              
              <input v-if="editedRow === transaction.transID"
              v-model="transaction.transaction_type"
              class="input-field"
              type="text"
              />
              <span v-else>{{ transaction.transaction_type }}</span></td>

            <td>{{ transaction.createdBy }}</td>
            <td>{{ new Date(transaction.createdDate).toLocaleString() }}</td>
            <td>{{ transaction.updatedBy }}</td>
            <td>{{ new Date(transaction.updatedDate).toLocaleString() }}</td>
            <td>
              <button v-if="editedRow === transaction.transID" @click="saveEdit(transaction)" class="btn save-btn">Save</button>
              <button v-else @click="editTransaction(transaction)" class="btn edit-btn">Edit</button>
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
  import api from '../APIs/TransactionAPI'; // Adjust the import path if needed
  
  export default {
    data() {
      return {
        transactions: [],
        currentPage: 1,
        pageSize: 10,
        editedRow: null,
        transactionForm: {
          transID: null,
          accountID: null,
          transTypeID: null,
          amount: 0,
          date: '',
          transaction_type: '',
          createdDate: '',
          createdBy: 'System',
          updatedDate: '',
          updatedBy: 'System',
        },
        isEdit: false,
       
      };
    },
    computed: {
      paginatedTransactions() {
        const start = (this.currentPage - 1) * this.pageSize;
        const end = start + this.pageSize;
        return this.transactions.slice(start, end);
      },
      totalPages() {
        return Math.ceil(this.transactions.length / this.pageSize);
      },
    },
    methods: 
    {
      toggleForm() {
        this.showForm = !this.showForm;
      },
      async fetchTransactions() {
        try {
          const response = await api.get('/');
          this.transactions = response.data;
        } catch (error) {
          console.error('Failed to load transactions:', error);
        }
      },
  
      async addTransaction() {
        try {
          const transactionData = {
            ...this.transactionForm,
            createdDate: this.transactionForm.createdDate || new Date().toISOString(),
          };
          const response = await api.post('/', transactionData);
          this.transactions.push(response.data);
          this.clearForm();
        } catch (error) {
          console.error('Failed to add transaction:', error.response?.data || error.message);
        }
      },
  
      async updateTransaction() {
        try {
          this.transactionForm.updatedBy = this.transactionForm.updatedBy || 'System';
          this.transactionForm.updatedDate = new Date().toISOString();
          await api.put(`/${this.transactionForm.transID}`, this.transactionForm);
          const index = this.transactions.findIndex(t => t.transID === this.transactionForm.transID);
          if (index !== -1) {
            this.transactions.splice(index, 1, { ...this.transactionForm });
          }
          this.clearForm();
        } catch (error) {
          console.error('Failed to update transaction:', error.response?.data || error.message);
        }
      },
  
      editTransaction(transaction) {
        this.editedRow = transaction.transID;
      },
  
      async saveEdit(transaction) {
        try {
          if (!transaction.updatedDate) {
            transaction.updatedDate = new Date().toISOString();
          }
          await api.put(`/${transaction.transID}`, transaction);
          this.editedRow = null;
        } catch (error) {
          console.error('Failed to save transaction edit:', error);
        }
      },
  
      clearForm() {
        this.transactionForm = {
          transID: null,
          accountID: null,
          transTypeID: null,
          amount: 0,
          date: '',
          transaction_type: '',
          createdDate: '',
          createdBy: 'System',
          updatedDate: '',
          updatedBy: 'System',
        };
        this.isEdit = false;
        this.showForm = false;
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
      this.fetchTransactions();
    },
  };
  </script>
  
  <style scoped>
/* Container styling */
.transaction-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

/* Title Styling */
.title {
  font-size: 2.2rem;
  font-weight: 600;
  text-align: center;
  margin-bottom: 20px;
  color: #333;
}

/* Form Styling */
.transaction-form {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.1);
  margin-bottom: 25px;
}

.form-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
}

.input-field, .select-field, .date-field {
  padding: 10px;
  border: 1px solid #dcdcdc;
  border-radius: 6px;
  width: 100%;
  font-size: 1rem;
  box-sizing: border-box;
}

textarea {
  min-height: 100px;
  resize: vertical;
}

.form-container .full-width {
  grid-column: span 2;
}

/* Button group */
.button-group {
  grid-column: span 2;
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

.btn {
  padding: 10px 15px;
  font-size: 1rem;
  border-radius: 4px;
  cursor: pointer;
  border: none;
  color: #fff;
  transition: background-color 0.3s ease;
}

.btn.primary {
  background-color: #28a745;
}

.btn.secondary {
  background-color: #dc3545;
}

.btn.save-btn {
  background-color: #007bff;
}

.btn.edit-btn {
  background-color: #ffc107;
}

.btn:hover {
  opacity: 0.9;
}

/* Table Styling */
.transaction-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

.transaction-table th, .transaction-table td {
  border: 1px solid #e0e0e0;
  padding: 12px;
  text-align: left;
}

.transaction-table th {
  background-color: #f2f2f2;
  font-weight: 600;
  color: #333;
}

.transaction-table td {
  background-color: #fff;
  word-wrap: break-word;
}

.transaction-table tr:hover {
  background-color: #f9f9f9;
}

/* Action button styling in table */
.table-action-btn {
  padding: 5px 10px;
  font-size: 0.9rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  color: #fff;
}

.edit-btn {
  background-color: #007bff;
}

.delete-btn {
  background-color: #dc3545;
}

.view-btn {
  background-color: #28a745;
}

/* Pagination styling */
.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
}

.pagination-button {
  padding: 10px 15px;
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.pagination-info {
  font-size: 1rem;
  color: #333;
  margin: 0 10px;
}

/* Mobile view */
@media (max-width: 768px) {
  .form-container {
    grid-template-columns: 1fr;
  }

  .transaction-container {
    padding: 10px;
  }

  .pagination-button, .btn {
    padding: 8px 12px;
    font-size: 0.9rem;
  }
}
</style>



import axios from 'axios';

// // Create an Axios instance with the base URL and timeout
// const TransactionAPI = axios.create({
//   baseURL: 'http://localhost:5113/api/transaction',
//   timeout: 2000,
// });

// // Exported functions for interacting with the Account API
// export const getAllTransactions = () => TransactionAPI.get('/');
// export const GetTransactionAsync = (id) => TransactionAPI.get(`/${id}`);
// export const AddTransactionAsync = (transaction) => TransactionAPI.post('/', transaction);
// export const UpdateTransactionAsync = (id, transaction) => TransactionAPI.put(`/${id}`, transaction);
// export const DeleteTransactionAsyn = (id) => TransactionAPI.delete(`/${id}`);

// export default TransactionAPI;













