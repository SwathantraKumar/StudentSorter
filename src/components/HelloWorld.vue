<template>
  <div>
    <h1>{{ msg }}</h1>

    <input type="file" @change="handleFileUpload">
    <h4>Bubble Sort Performance in milliseconds {{ fetchedData.bubbleSortPerformance }}</h4>
    <h4>Merge Sort Performance in milliseconds {{ fetchedData.mergeSortPerformance }}</h4>
    <h4>Heap Sort Performance in milliseconds {{ fetchedData.heapSortPerformance }}</h4>

    <table v-if="fetchedData.sortedStudents && fetchedData.sortedStudents.length > 0" id="sortedTable">
      <thead>
        <tr>
          <th>Name</th>
          <th>Performance</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in fetchedData.sortedStudents" :key="item.name">
          <td>{{ item.name }}</td>
          <td>{{ item.performance }}</td>
        </tr>
      </tbody>
    </table>

    <button @click="saveTableAsFile">Save as File</button>


  </div>
</template>

<script>
import axios from 'axios';
import Papa from 'papaparse';
import { saveAs } from 'file-saver';


export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      fetchedData: []
    };
  },
  methods: {
    handleFileUpload(event) {
      const file = event.target.files[0];
      Papa.parse(file, {
        complete: (result) => {
          const csvData = result.data;
          const jsonData = this.convertCSVtoJSON(csvData);
          const jsonArray = Object.values(jsonData);
          this.convertCSVtoJSON(csvData);
          this.processCSVData(jsonArray)
        }
      });
    },

    convertCSVtoJSON(csvData) {
      const headers = csvData[0];
      const jsonData = [];

      for (let i = 1; i < csvData.length; i++) {
        const row = csvData[i];
        const obj = {};

        for (let j = 0; j < headers.length; j++) {
          obj[headers[j]] = row[j];
        }

        jsonData.push(obj);
      }

      return jsonData;
    },


    processCSVData(jsonArray) {
      axios.post('http://localhost:9090/api/students/sort', jsonArray)
        .then(response => {
          console.log(response.data);
          this.fetchedData = response.data;
        })
        .catch(error => {
          console.error(error);
        });
    },
    saveTableAsFile() {
      let csvData = '';

      const table = document.getElementById('sortedTable');
      const headers = Array.from(table.querySelectorAll('th')).map(header => header.innerText);
      csvData += headers.join(',') + '\n';

      const rows = Array.from(table.querySelectorAll('tr'));
      rows.forEach(row => {
        const rowData = Array.from(row.querySelectorAll('td')).map(cell => cell.innerText);
        csvData += rowData.join(',') + '\n';
      });

      const blob = new Blob([csvData], { type: 'text/csv;charset=utf-8' });
      saveAs(blob, 'sorted.csv');
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
