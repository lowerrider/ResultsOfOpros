<template>
  <div>
    <h1>Итоги победителей номинаций</h1>
    <table v-if="nominations.length">
      <thead>
        <tr>
          <th>НАСЛЕДИЕ ЮНЕСКО</th>
          <th>ЗАГАДОЧНЫЙ МАЙЯ</th>
          <th>ТЕКИЛА ЛЮБОВЬ</th>
          <th>ОСТРЫЙ КАК ПЕРЕЦ ЧИЛИ</th>
          <th>ГРЯЗНЫЙ САНЧЕС</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>
            <ul>
              <li v-for="member in sortedNominations(0)" :key="member.personID">
                {{ member.title }} ({{ member.votes }})
              </li>
            </ul>
          </td>
          <td>
            <ul>
              <li v-for="member in sortedNominations(1)" :key="member.personID">
                {{ member.title }} ({{ member.votes }})
              </li>
            </ul>
          </td>
          <td>
            <ul>
              <li v-for="member in sortedNominations(2)" :key="member.personID">
                {{ member.title }} ({{ member.votes }})
              </li>
            </ul>
          </td>
          <td>
            <ul>
              <li v-for="member in sortedNominations(3)" :key="member.personID">
                {{ member.title }} ({{ member.votes }})
              </li>
            </ul>
          </td>
          <td>
            <ul>
              <li v-for="member in sortedNominations(4)" :key="member.personID">
                {{ member.title }} ({{ member.votes }})
              </li>
            </ul>
          </td>
        </tr>
      </tbody>
    </table>
    <p v-else>Загрузка данных...</p>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";

const nominations = ref([]);

const fetchData = async () => {
  try {
    const membersResponse = await axios.get(
      "https://b5862cf2cca63d34.mokky.dev/members"
    );
    const answersResponse = await axios.get(
      "https://b5862cf2cca63d34.mokky.dev/answers"
    );

    const members = membersResponse.data;
    const answers = answersResponse.data;

    const nominationKeys = {
      unesko: "НАСЛЕДИЕ ЮНЕСКО",
      maya: "ЗАГАДОЧНЫЙ МАЙЯ",
      tekila: "ТЕКИЛА ЛЮБОВЬ",
      chily: "ОСТРЫЙ КАК ПЕРЕЦ ЧИЛИ",
      derty: "ГРЯЗНЫЙ САНЧЕС",
    };

    const votesCount = {
      unesko: {},
      maya: {},
      tekila: {},
      chily: {},
      derty: {},
    };

    answers.forEach((answer) => {
      Object.entries(answer).forEach(([key, value]) => {
        if (key in votesCount) {
          const nomineeId = value; // personId
          if (!votesCount[key][nomineeId]) {
            votesCount[key][nomineeId] = 0;
          }
          votesCount[key][nomineeId]++; // Увеличиваем количество голосов
        }
      });
    });

    Object.keys(nominationKeys).forEach((key) => {
      const nomineesList = members.map((member) => {
        const votes = votesCount[key][member.personId] || 0;
        return { title: member.title, personID: member.personId, votes };
      });

      nominations.value.push({
        title: nominationKeys[key],
        nominees: nomineesList,
      });
    });
  } catch (error) {
    console.error("Ошибка при загрузке данных:", error);
  }
};

// Функция для фильтрации номинантов
const filteredNominations = (index) => {
  return nominations.value[index].nominees.filter((member) => member.votes > 0);
};

// Функция для сортировки номинантов по количеству голосов
const sortedNominations = (index) => {
  return filteredNominations(index).sort((a, b) => b.votes - a.votes);
};

onMounted(fetchData);
</script>

<style>
table {
  width: 100%;
  border-collapse: collapse;
}
th,
td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: center;
}
th {
  background-color: #f2f2f2;
}
ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
}
</style>
