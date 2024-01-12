<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios';

const transcript = ref('')
const isRecording = ref(false)

const Recognition = window.SpeechRecognition || window.webkitSpeechRecognition
const sr = new Recognition()

onMounted(() => {
    sr.continuous = true
    sr.interimResults = true

    sr.onstart = () => {
        console.log('SR Started')
        isRecording.value = true
    }

    sr.onend = () => {
        console.log('SR Stopped')
        isRecording.value = false
    }

    sr.onresult = (evt) => {
        for (let i = 0; i < evt.results.length; i++) {
            const result = evt.results[i]

            if (result.isFinal) CheckForCommand(result)
        }

        const t = Array.from(evt.results)
            .map(result => result[0])
            .map(result => result.transcript)
            .join('')

        transcript.value = t
    }
})

const CheckForCommand = (result) => {
    const t = result[0].transcript;
    if (t.includes('stop recording')) {
        sr.stop()
    } else if (
        t.includes('what is the time') ||
        t.includes('what\'s the time')
    ) {
        sr.stop()
        alert(new Date().toLocaleTimeString())
        setTimeout(() => sr.start(), 100)
    }
}

const ToggleMic = () => {
    if (isRecording.value) {
        sr.stop()
    } else {
        sr.start()
    }
}

const ToggleMicStop = () => {
    sr.stop()
}

const ToggleClearTranscript = () => {
    transcript.value = ''; // Membersihkan nilai transcript
}

const saveTranscriptToApi = () => {
	console.log(transcript)

    const apiUrl = 'http://127.0.0.1:8000/api/save-transcript'; // Ganti dengan URL endpoint API yang sesuai
    const dataToSend = {
        transcript: transcript.value,
    };

    axios.post(apiUrl, dataToSend)
        .then(response => {
            console.log('Data saved successfully', response.data);
            // Tambahkan logika atau notifikasi sesuai kebutuhan
        })
        .catch(error => {
            console.error('Error saving data', error);
            // Tambahkan logika atau notifikasi kesalahan
        });
};
</script>

<template>
    <div class="app">
        <button :class="`mic`" @click="ToggleMic">Record</button>
        <button :class="`mic`" @click="ToggleMicStop">Stop</button><br>
        <button :class="`mic`" @click="ToggleClearTranscript">Clear</button><br>
        <div>
            <textarea class="transcript" v-text="transcript"></textarea>
        </div>
        <button type="button" @click="saveTranscriptToApi">Save</button>
    </div>
</template>


<style>
* {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
	font-family: 'Fira sans', sans-serif;
}

textarea {
	height: 200px;
	width: 50%;
	
}

body {
	background: #281936;
	color: #FFF;
}
</style>
