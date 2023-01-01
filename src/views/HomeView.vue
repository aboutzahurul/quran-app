<script setup>

import axios from 'axios';
import { ref, onMounted, onBeforeMount, computed } from 'vue';

let surahList = ref();
let currentSurah = ref();
let suraDataAr = ref();
let surahDataBn = ref();
let currentAyahTracks = ref([]);
let currentAyahNumbers = ref([]);
let currentAyahNumber = ref();
let firstAyahAudio = ref();
let audioLoop = ref();
let surahReloaded = ref();
let pageLoading = ref();
let reciterList = ref();
let currentReciter = ref();

onBeforeMount(() => {

	// Setting initial values
	currentSurah.value = getCurrentSurah();
	currentReciter.value = getCurrentReciter();
	currentAyahNumber.value = 1;
	firstAyahAudio.value = 'https://cdn.islamic.network/quran/audio/128/ar.alafasy/1.mp3';
	audioLoop.value = true;
	surahReloaded.value = false;
	pageLoading.value = true;
})

onMounted(() => {
	// Preloading list of surah
	LoadSurahList();

	// Preloading list of reciter
	LoadReciterList();

	// Preloading first ayah
	loadFirstAyahAudio();

	// initial first surah load
	querySurah();
})

/**
 * Setting current surah number to storage
 * @param {int} currentSurah
 */
function setCurrentSurah( currentSurah ){
	localStorage.setItem('currentSurah', currentSurah);
}

/**
 * Getting current surah selection from storage
 */
function getCurrentSurah(){
	let currentSurah = localStorage.getItem('currentSurah');
	return currentSurah ? currentSurah : 1;
}

/**
 * Setting current reciter to storage
 * @param {string} currentReciter
 */
function setCurrentReciter( currentReciter ){
	localStorage.setItem('currentReciter', currentReciter);
}

/**
 * Getting current reciter selection from storage
 */
function getCurrentReciter(){
	let currentReciter = localStorage.getItem('currentReciter');
	return currentReciter ? currentReciter : 'ar.alafasy';
}

/**
 * Loading list of surah
 */
function LoadSurahList(){
	axios.get('https://api.alquran.cloud/v1/surah')
		.then((response) => {
			surahList.value = response.data.data;
		});
}

/**
 * Loading list of reciter
 */
function LoadReciterList(){
	axios.get('https://api.alquran.cloud/v1/edition/format/audio')
		.then((response) => {
			reciterList.value = response.data.data;
		});
}

// Loading first ayah (bismillah) audio of current reciter
function loadFirstAyahAudio(){
	axios.get('https://api.alquran.cloud/v1/ayah/1/' + currentReciter.value)
		.then((response) => {
			firstAyahAudio.value = response.data.data.audio;
		});
}

/**
 * getting surah number on select change
 */
function handleSurahListChange(e) {
	currentSurah.value = e.target.value;
	setCurrentSurah(e.target.value);
	querySurah();
}

/**
 * getting reciter identifier on select change
 */
function handleReciterListChange(e) {
	currentReciter.value = e.target.value;
	setCurrentReciter(e.target.value);
	loadFirstAyahAudio();
	querySurah();
}

/**
 * getting surah details by surah number
 */
function querySurah() {

	surahReloaded.value = true;
	pageLoading.value = true;

	axios.get('https://api.alquran.cloud/v1/surah/' + currentSurah.value + '/editions/' + currentReciter.value + ',bn.bengali')
		.then((response) => {

			suraDataAr.value = response.data.data[0];
			surahDataBn.value = response.data.data[1];

			// clear old values before load new
			currentAyahTracks.value = [];
			currentAyahNumbers.value = [];

			// looping response to collet ayah
			response.data.data[0].ayahs.forEach(function (item, index) {
				// console.log(item.number);
				// currentAyahTracks.value[index] = item.audio;
				currentAyahTracks.value[item.number] = item.audio;
				currentAyahNumbers.value[index] = item.number;
			});

			// reset player as new track loaded
			resetPlayer();

			pageLoading.value = false;

		});
}

/**
 * call once player ended a track for next audio if any
 */
function onAudioTrackEnd(e) {

	let audio = document.getElementById('audioPlayer');
	let currentlyPlaying = audio.getAttribute('data-current');

	if(currentlyPlaying == 1 && currentAyahNumbers.value[0] == 1){
		currentlyPlaying = currentAyahNumbers.value[1];
	} else {
		if(currentlyPlaying == 1 ){
			currentlyPlaying = currentAyahNumbers.value[0];
		}else{
			++currentlyPlaying;
		}
	}

	if (currentAyahTracks.value[currentlyPlaying]) {
		audio.setAttribute('data-current', currentlyPlaying);
		audio.src = currentAyahTracks.value[currentlyPlaying];
		audio.load();
		audio.play();
	} else {
		resetPlayer();
	}
	highlightAyah();
}

/**
 * reset player
 */
function resetPlayer() {
	let audio = document.getElementById('audioPlayer');
	if(audio){
		audio.setAttribute('data-current', 1);
		currentAyahNumber.value = 1;
		audio.src = firstAyahAudio.value;
		audio.load();

		if(audioLoop.value == true && surahReloaded.value == false){
			audio.play();
		} else {
			audio.pause();
		}
	}
}

/**
 * Run while audio is playing
 */
function onAudioPlaying(e){

	let audio = document.getElementById('audioPlayer');
	currentAyahNumber.value = audio.getAttribute('data-current');
	surahReloaded.value = false;
	highlightAyah();
}

/**
 * Highlight ayah based on what currently playing
 */
function highlightAyah(){

	document.querySelectorAll('.ayah').forEach(function(ayahElement) {
		ayahElement.classList.replace('ayah-playing','ayah-played');
	});
	
	let currentAyah = document.getElementById(currentAyahNumber.value);
	if(currentAyah){
		currentAyah.classList.add('ayah-playing');
		currentAyah.scrollIntoView({
			behavior: 'smooth'
		});
	} else {
		// doing for last ayah
		document.querySelectorAll('.ayah').forEach(function(ayahElement) {
			ayahElement.classList.replace('ayah-playing','ayah-played');
		});
	}
}

/**
 * Handle loop selection checkbox
 */
function handleLoop(e){
	// console.log(e.currentTarget.checked);
	audioLoop.value = e.currentTarget.checked;
}

</script>

<template>
	<main>

		<div class="fixed left-0 bottom-0 w-full bg-white z-10 border-t shadow-[0_35px_60px_15px_rgba(0,0,0,0.4)]">
			<div class="max-w-7xl px-6 py-2 flex justify-between items-center mx-auto flex-col md:flex-row">

			<select @change="handleReciterListChange" name="surah-select"
				class="mt-1 block rounded-md border border-gray-300 bg-white py-2 px-3 shadow-sm focus:border-indigo-500 focus:outline-none focus:ring-indigo-500 sm:text-sm">
				<template v-for="reciter in reciterList">
					<option v-if="reciter.language == 'ar'" :value="reciter.identifier" :selected="reciter.identifier == currentReciter ? true : false">
						{{ reciter.englishName }}
					</option>
				</template>
			</select>

			<select @change="handleSurahListChange" name="surah-select"
				class="mt-1 block rounded-md border border-gray-300 bg-white py-2 px-3 shadow-sm focus:border-indigo-500 focus:outline-none focus:ring-indigo-500 sm:text-sm">
				<option v-for="surah in surahList" :value="surah.number" :selected="surah.number == currentSurah ? true : false">
					[{{ surah.number }}] {{ surah.name }} ({{ surah.englishName }})
				</option>
			</select>

			<audio v-if="currentAyahTracks[currentAyahNumbers[0]]" v-on:ended="onAudioTrackEnd" @play="onAudioPlaying" :data-current="currentAyahNumber" id="audioPlayer" controls>
				<source :src="firstAyahAudio" type="audio/mpeg">
			</audio>


			<div class="flex items-start">
				<div class="flex h-5 items-center">
					<input :checked="audioLoop ? true : false" @change="handleLoop" id="audioLoop" name="audioLoop" type="checkbox" class="h-4 w-4 rounded border-gray-300 text-emerald-600 focus:ring-emerald-600">
				</div>
				<div class="ml-3 text-sm">
					<label for="audioLoop" class="font-medium text-gray-700">Loop Audio</label>
				</div>
			</div>

			</div>
		</div>

		<div class="relative border border-gray-200 px-8 py-8 bg-slate-100">
			<div v-if="suraDataAr" class="flex justify-center text-xs sm:text-base mb-4 pb-4">
				<p class="px-0">[{{ suraDataAr.englishName }}]</p>
				<p class="px-2">[{{ suraDataAr.englishNameTranslation }}]</p>
				<p class="px-0">[{{ suraDataAr.name }}]</p>
			</div>
			<p class="text-4xl text-center">بِسْمِ ٱللَّهِ ٱلرَّحْمَٰنِ ٱلرَّحِيمِ</p>
		</div>

		<div class="relative">

			<div v-if="pageLoading" class="h-[50vh] grid place-items-center border border-t-0">
				<div>
					<div class="spinner"></div>
					<p class="text-lg text-gray-400">Loading...</p>
				</div>
			</div>

			<div v-if="suraDataAr && pageLoading == false" class="relative border border-gray-200 p-0 sm:p-8 border-t-0" id="ayahs-holder">
	
				<div v-for="(ayah, rowindex) in suraDataAr.ayahs" :key="ayah.number" :id="ayah.number"
					class="ayah border-b border-gray-200 py-6 last-of-type:border-0 text-gray-600 duration-500">
					<p class="text-2xl text-center px-4">
						<span class="text-sm border rounded-full px-2 py-1 inline-block">{{ ayah.numberInSurah }}</span> {{ ayah.text }}
					</p>
					<p class="text-xl text-center px-4">
						{{ surahDataBn.ayahs[rowindex].text }}
					</p>
				</div>
	
			</div>

		</div>

		<div class="gap h-40 md:h-12"></div>

	</main>
</template>

<style>
.ayah-played{
    background-color: #fbfbfb;
}
.ayah-playing{
    background-color: #f9f9f9;
	scroll-margin-top: 80px;
	color: #000 !important;
	box-shadow: inset 0 0 20px rgba(0,0,0,0.1);
	text-shadow: 0 1px 1px rgba(0,0,0,0.3);
}
footer{
	display: none;
}
</style>