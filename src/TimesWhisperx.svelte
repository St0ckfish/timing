<!-- can likely be deleted -->
<audio {src} bind:this={audio} controls on:timeupdate={timeupdate}/>

<fieldset class="p-1 mt-4">
  <label class=block>Times <textarea class="block" on:input={e => set_times(e.target.value)} on:paste={paste}/></label>
  <button class="btn !py-1" use:filedrop on:filedrop={e => on_select(e.detail.files.accepted[0])}>Select <icon id=open></button>
  <!-- <button class=btn on:click={copy_times}><icon id=copy></button> -->
</fieldset>

{#if times}
{times.word_segments[current_i].word}
<div class=whitespace-pre-wrap>
  <!-- {times.segments[0].text} -->
</div>
{/if}

<script>
import {read_file} from 'components/src/util.js'
import {get_ayah_id} from '~/util/mushaf.js'
import {filedrop} from 'filedrop-svelte'

const pad = i => i.toString().padStart(3, '0')
async function on_select(file) {
    set_times((await read_file(file)).trim())
}

let mushaf
// fetch('https://fonts.nuqayah.com/quran.txt').then(r => r.text()).then(text => {
//     mushaf = text.split('\n').map(l => l.split(',')[0])
// })
let audio
let times
/* 
"word_segments":
[
    {
        "word": "بَعَثَهُ",
        "start": 0.14,
        "end": 1.02,
        "score": 0.889
    },
    {
        "word": "الرحمنُ",
        "start": 1.04,
        "end": 1.8,
        "score": 0.74
    },
  ]
*/
let current_i = 0
function timeupdate() {
    // loop over word_segments, and display the current playing word
    const time = audio.currentTime
    current_i = 0
    while (current_i < times.word_segments.length && (!times.word_segments[current_i].end || times.word_segments[current_i].end < time)) current_i++
    console.log({current_i})
}

let text = ''
function set_times(val) {
    // times = val.split('\n').map(v => +v)
    times = JSON.parse(val)
    // times = {word_segments: times.flatMap(x => x.words)}
    // text = times.map
    window.t = times
}
// function copy_times() {
//     // Sura,Ayah,IndexStart,IndexEnd,TimeStart,TimeEnd
//     let times_filtered = times.filter(t => t[1] !== '0')
//     times_filtered = times_filtered.map(t => [...(times_filtered.length === 6236 ? [] : t.slice(0, 2)), ...t.slice(4, 6)].map(i => +i))
//     // TODO: avg times
//     const text = JSON.stringify(times_filtered).replaceAll('],[', '],\n[')
//     navigator.clipboard.writeText(text)
// }

// $: if (audio) audio.currentTime = ((times[get_ayah_id(surah, ayah) - 1] || 0) - (surah === 2 ? 0 : 0)) / 1000
let src = '/akhlaq.mp3'
// let src = ''
// $: {
//   src = audio_base + pad(surah) + '.' + ext
//   audio?.play()
// }
function paste(e) {
    set_times(e.clipboardData.getData('text/plain').trim())
    e.preventDefault() // avoid forcing the browser to render the text as it can be large and thus hang
}
</script>
