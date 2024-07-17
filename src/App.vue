<template>
	<main class="app">
		<h1>The Quiz</h1>

		<section v-if="loading">
			<p>Đang tải câu hỏi... <br>Nếu trong quá trình tải câu hỏi lâu thì hãy kiểm tra lại đường truyền internet<br>Xin Cảm Ơn!!!</p>
		</section>

		<section class="quiz" v-else-if="!quizCompleted && questions.length > 0">
			<div class="quiz-info">
				<!-- Updated line to render question with v-html -->
				<span class="question" v-html="decodeEntities(getCurrentQuestion.question)"></span>
				<span class="score">Điểm {{ score }}/{{ questions.length }}</span>
			</div>

			<!-- Replacing the options section -->
			<div class="options">
				<label v-for="(option, index) in getCurrentQuestion.options" :for="'option' + index" :class="`option ${getCurrentQuestion.selected == index
					? index == getCurrentQuestion.correct_answer
						? 'correct'
						: 'wrong'
					: ''
					} ${getCurrentQuestion.selected != null && index != getCurrentQuestion.selected
						? 'disabled'
						: ''
					}`">
					<input type="radio" :id="'option' + index" :name="getCurrentQuestion.index" :value="index"
						v-model="getCurrentQuestion.selected" :disabled="getCurrentQuestion.selected" @change="SetAnswer" />
					<span>{{ decodeEntities(option) }}</span> <!-- Update to decode option entities -->
				</label>
			</div>

			<button @click="NextQuestion" :disabled="!getCurrentQuestion.selected">
				{{
					getCurrentQuestion.index === questions.length - 1
					? 'Hoàn Thành!'
					: getCurrentQuestion.selected === null
						? 'Hãy Chọn Câu Trả Lời'
						: 'Câu Hỏi Tiếp Theo'
				}}
			</button>
		</section>

		<section v-else>
			<h2>Chúc Mừng! Bạn Đã Hoàn Thành Chương Trình Câu Đố</h2>
			<p>Điểm Của Bạn: {{ score }}/{{ questions.length }}</p>
			<button class="restart-btn" @click="restartQuiz">Chơi Lại</button>
		</section>

		<div class="footer">
		<h1>&copy; Mọi Bản Quyền Thuộc Về <span>Huỳnh Vĩnh Tiến</span></h1>
		<img src="../public/logo_1.png"/>
		</div>

	</main>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';

const loading = ref(true);
const questions = ref([]);
const quizCompleted = ref(false);
const currentQuestion = ref(0);

const fetchData = async () => {
	try {
		const response = await axios.get('https://opentdb.com/api.php?amount=10');
		const data = response.data;
		questions.value = data.results.map((question, index) => {
			const options = [...question.incorrect_answers, question.correct_answer].sort(() => Math.random() - 0.5);
			return {
				...question,
				options,
				selected: null,
				correct_answer: options.indexOf(question.correct_answer), // Assign the correct index
				index,
			};
		});
		loading.value = false;
	} catch (error) {
		console.error('Error fetching questions:', error);
		loading.value = false;
	}
};

// Method to decode HTML entities
const decodeEntities = (html) => {
	const txt = document.createElement('textarea');
	txt.innerHTML = html;
	return txt.value;
};

const score = computed(() => {
	let value = 0;
	questions.value.forEach((q) => {
		// Using String comparison to handle type mismatch between correct_answer and selected
		if (q.selected !== null && String(q.correct_answer) === String(q.selected)) {
			value++;
		}
	});
	return value;
});

const getCurrentQuestion = computed(() => {
	return questions.value[currentQuestion.value];
});

const SetAnswer = (e) => {
	questions.value[currentQuestion.value].selected = e.target.value;
};

const NextQuestion = () => {
	if (currentQuestion.value < questions.value.length - 1) {
		currentQuestion.value++;
		return;
	}

	quizCompleted.value = true;
};

const restartQuiz = () => {
    loading.value = true;
    quizCompleted.value = false;
    currentQuestion.value = 0;
    fetchData();
};

onMounted(() => {
	fetchData();
});
</script>
  
<style src="./style.css"></style>
