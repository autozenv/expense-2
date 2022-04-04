<template>
    <div id="app">
        <h1 class="title">EXPENSE TRACKER</h1>
        <p class="abt">
            <q>This expense tracker helps you track your expenses for as long as you wish. Whenever you wish to start
                tracking afresh (This can be after a week or a month or any period of time), click on the clear records
                button to clear everything. Click on the 'Export as CSV button to export the records as a .csv
                file'
            </q><small> -- Manaswini</small>
        </p>
        <div class="flex">
            <div class="flex_content flex_form">
                <form @submit.prevent="addExpense" class="form">
                    <label for="amount">Amount</label>
                    <input type="number" id="amount" v-model.number="amount" autocomplete="off" required>

                    <label for="to">To</label>
                    <input type="text" id="to" v-model="to" autocomplete="off" required>

                    <label for="note">Note</label>
                    <textarea id="note" rows="5" v-model="info"></textarea>

                    <label for="date">Date</label>
                    <input type="date" id="date" v-model="date" required>

                    <button class="btn btn_success" type="submit">Add expense</button>
                    <button class="btn btn_danger" @click.prevent="clearExpenses">Clear record</button>
                    <button class="btn btn_primary" @click.prevent="getCsv">Export as .csv</button>
                </form>
            </div>
            <div class="flex_content flex_expenses">
                <h2 class="head">Expenses</h2>
                <div class="total">
                    <h3>TOTAL</h3>
                    <h3 v-if="total > 0 && Math.abs(total) !== Math.floor(total)">${{ total }}</h3>
                    <h3 v-if="Math.abs(total) === Math.floor(total)">${{ total }}.0</h3>
                    <h3 v-else>$0.0</h3>
                </div>
                <div class="expense" v-for="expense in expenses">
                    <p class="to"><b>To:</b> {{ expense.to }}</p>
                    <p class="date"><b>Date:</b> {{ expense.date }}</p>
                    <p class="amnt"><b>Amount:</b> ${{ expense.amount }}</p>
                    <p class="note"><b>Note:</b> {{ expense.info }}</p>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default{
    el: "#app",
    data: function () {
            return{
            amount: '',
            to: '',
            info: '',
            date: '',
            total: 0,
            error: false,
            expenses: []
        }
    },
    methods: {
        addExpense: function () {
            if (this.checkForm()) {
                this.error = true;
            } else {
                const data = {
                    amount: this.amount,
                    to: this.to,
                    info: this.info,
                    date: this.date
                };
                this.amount = '';
                this.to = '';
                this.info = '';
                this.date = '';
                this.expenses.unshift(data);
                localStorage.setItem('q-vue-expenses', JSON.stringify(this.expenses));
                this.getTotal()
            }
        },
        clearExpenses: function () {
            this.expenses = [];
            this.total = 0;
            localStorage.setItem('q-vue-expenses', JSON.stringify(this.expenses));

        },
        getTotal: function () {
            let initialTotal = 0;
            this.expenses.forEach(data => {
                initialTotal += Math.abs(data.amount)
            });
            this.total = initialTotal;
        },
        checkForm: function () {
            return this.amount === '' || this.to === '' || this.info === '' || this.data === '' ? true : false
        },
        getCsv: function () {
            const headers = {
                amount: "Amount",
                to: "For",
                info: "Info",
                date: "Date"
            };

            let itemsNotFormatted = JSON.parse(localStorage.getItem('q-vue-expenses'));

            let itemsFormatted = [];

            // format the data
            itemsNotFormatted.forEach((item) => {
                itemsFormatted.push({
                    amount: item.amount, // remove commas to avoid errors,
                    to: item.to,
                    info: item.info,
                    date: item.date
                });
            });

            const fileTitle = 'Expenses'; // or 'my-unique-title'

            this.exportCSVFile(headers, itemsFormatted, fileTitle);
        },
        convertToCSV: function (objArray) {
            var array = typeof objArray != 'object' ? JSON.parse(objArray) : objArray;
            var str = '';

            for (var i = 0; i < array.length; i++) {
                var line = '';
                for (var index in array[i]) {
                    if (line != '') line += ','

                    line += array[i][index];
                }

                str += line + '\r\n';
            }

            return str;
        },
        exportCSVFile: function (headers, items, fileTitle) {
            if (headers) {
                items.unshift(headers);
            }

            // Convert Object to JSON
            var jsonObject = JSON.stringify(items);

            var csv = this.convertToCSV(jsonObject);

            var exportedFilenmae = fileTitle + '.csv' || 'export.csv';

            var blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
            if (navigator.msSaveBlob) { // IE 10+
                navigator.msSaveBlob(blob, exportedFilenmae);
            } else {
                var link = document.createElement("a");
                if (link.download !== undefined) { // feature detection
                    // Browsers that support HTML5 download attribute
                    var url = URL.createObjectURL(blob);
                    link.setAttribute("href", url);
                    link.setAttribute("download", exportedFilenmae);
                    link.style.visibility = 'hidden';
                    document.body.appendChild(link);
                    link.click();
                    document.body.removeChild(link);
                }
            }
        }
    },
    created: function () {
        if (localStorage.getItem('q-vue-expenses')) {
            this.expenses = JSON.parse(localStorage.getItem('q-vue-expenses'));
        } else {
            localStorage.setItem('q-vue-expenses', JSON.stringify([]));
        }
        this.getTotal()
    }
}
</script>

<style>
    body {
    background-color: rgb(38, 76, 126);
    padding: 0 50px;
}

#app {
    font-family: "Nunito", sans-serif;
    color: aliceblue;
}

#app .title,
#app .abt {
    text-align: center;
}

.flex {
    display: flex;
    width: 80%;
    margin: auto;
    justify-content: safe;
}

.flex_content {
    flex: 1;
    border: 1px solid #fff;
    border-radius: 7px;
}

.flex_content:nth-of-type(1) {
    margin-right: 1%;
}

.flex_content:nth-of-type(2){
    flex: 1.3;
}

.flex_content:nth-of-type(1){
    background-color: rgba(23, 47, 78, 0.918);
}

.form input {
    font-family: "Nunito", sans-serif;
    display: block;
    margin-bottom: 20px;
    margin-top: 4px;
    width: 95%;
    height: 30px;
    outline: none;
    border: 1px solid #ffffff71;
    border-radius: 4px;
    padding-left: 5%;
}

.form textarea {
    font-family: "Nunito", sans-serif;
    display: block;
    margin-bottom: 20px;
    margin-top: 4px;
    width: 95%;
    outline: none;
    border: 1px solid #ffffff71;
    border-radius: 4px;
    padding: 5px 0 5px 5%;
}

.flex_form{
    padding: 40px;
}

.form button{
    display: block;
    margin-bottom: 10px;
    width: 100%;
    height: 30px;
    text-transform: uppercase;
    font-weight: 400;
    outline: none;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-family: "Nunito", sans-serif;
}

form button.btn_success{
    background-color: rgb(3, 211, 90);
    color: #ffffff;
}

form button.btn_danger{
    background-color: rgb(228, 29, 29);
    color: #ffffff;
}

form button.btn_primary{
    background-color: rgb(6, 85, 255);
    color: #ffffff;
}

.flex_expenses{
    overflow-y: auto;
    max-height: 120vh;
}

.flex_expenses .head{
    background-color: rgba(23, 47, 78, 0.918);
    margin: 0;
    padding-top: 10px;
    padding-bottom: 4px;
    text-align: center;
    border-top-right-radius: 7px;
    border-top-left-radius: 7px;
}

.flex_expenses .total{
    background-color: rgba(23, 47, 78, 0.562);
    display: flex;
    padding: 5px 10px;
    justify-content: space-between;
    margin-bottom: 5%;
}

.flex_expenses .total h3{
    margin: 0;
}

.flex_expenses .expense{
    border-left: 10px solid rgba(23, 47, 78, 0.918);
    color: #000000;
    padding: 2%;
    border-radius: 7px;
    background-color: #fefefe;
    width: 85%;
    margin: auto;
    margin-bottom: 2%;
}

.flex_expenses .expense p{
    margin: 1%;
}

@media screen and (max-width: 768px){
    .flex{
        flex-direction: column;
        width: 90%;
    }

    .flex_expenses{
        margin-top: 3%;
    }
}

@media screen and (max-width: 500px){
    body{
        padding: 0 20px;
    }
}
</style>