<template>
  <div class="container">
    <h1>🎄 Итоги победителей номинаций 🎄</h1>
    <table v-if="nominations.length" class="nominations-table">
      <thead>
        <tr>
          <th>НАСЛЕДИЕ ЮНЕСКО</th>
          <th>ЗАГАДОЧНЫЙ МАЙЯ</th>
          <th>ТЕКИЛА ЛЮБОВЬ</th>
          <th>ОСТРЫЙ КАК ПЕРЕЦ ЧИЛИ</th>
          <th>CЛАВНЫЙ САНЧЕС</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>
            <ul>
              <li
                v-for="member in sortedNominations(0)"
                :key="member.personID"
                class="nominee"
              >
                <span class="name">{{ member.title }}</span>
                <span class="votes">{{ member.votes }}</span>
              </li>
            </ul>
          </td>
          <td>
            <ul>
              <li
                v-for="member in sortedNominations(1)"
                :key="member.personID"
                class="nominee"
              >
                <span class="name">{{ member.title }}</span>
                <span class="votes">{{ member.votes }}</span>
              </li>
            </ul>
          </td>
          <td>
            <ul>
              <li
                v-for="member in sortedNominations(2)"
                :key="member.personID"
                class="nominee"
              >
                <span class="name">{{ member.title }}</span>
                <span class="votes">{{ member.votes }}</span>
              </li>
            </ul>
          </td>
          <td>
            <ul>
              <li
                v-for="member in sortedNominations(3)"
                :key="member.personID"
                class="nominee"
              >
                <span class="name">{{ member.title }}</span>
                <span class="votes">{{ member.votes }}</span>
              </li>
            </ul>
          </td>
          <td>
            <ul>
              <li
                v-for="member in sortedNominations(4)"
                :key="member.personID"
                class="nominee"
              >
                <span class="name">{{ member.title }}</span>
                <span class="votes">{{ member.votes }}</span>
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
          votesCount[key][nomineeId] = (votesCount[key][nomineeId] || 0) + 1;
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
body {
  font-family: "Arial", sans-serif;
  background-color: #f0f8ff; /* Легкий голубой фон */
  color: #333;
  margin: 0;
  padding: 20px;
  background-image: url("https://example.com/snowflakes.png"); /* Замените на URL снежинок */
  background-size: cover;
}

h1 {
  text-align: center;
  color: #ff4757; /* Красный цвет для заголовка */
  font-family: "Georgia", serif; /* Элегантный шрифт */
}

.nominations-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
  background-color: rgba(255, 255, 255, 0.8); /* Полупрозрачный фон */
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

th {
  background-color: #ff4757; /* Красный фон для заголовков */
  color: white;
  padding: 12px;
}

td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
  vertical-align: top;
}

ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

.nominee {
  display: flex;
  justify-content: space-between;
  padding: 5px;
  border-bottom: 1px solid #ddd;
  transition: background-color 0.3s;
}

.nominee:hover {
  background-color: #ffefef; /* Светло-красный фон при наведении */
}

.name {
  font-weight: bold;
}

.votes {
  color: #ff4757; /* Красный цвет для голосов */
}
</style>
