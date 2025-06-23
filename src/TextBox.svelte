<fieldset class="p-1 mt-4">
  <div class=flex>
    <label class=block>times <input type=text on:paste={set_times} class="block"></label>
    <label class=block>sentences <input type=text on:paste={set_sentences} class="block"></label>
  </div>
</fieldset>

{#if sentences.length}
  <style bind:this={style_el}>
    [data-sentence-id="0"] {color: brown;}
    [data-word-id="0-0"] {font-weight: bold;}
  </style>
  <div class="whitespace-pre-wrap max-h-[70vh] overflow-auto" dir="auto" on:click={text_click}>
    <pre>{cur_time} - {current_s_i} - {current_w_i}</pre>
    {@html sentences.map((words, s_i) => {
      return `${s_i + 1}- <span data-sentence-id=${s_i + 1} title=${times[s_i]}>${
        words.map((w, w_i) => `<span title=${times[s_i][2][w_i]} data-word-id=${s_i + 1}-${w_i + 1}>${w}</span>`).join(' ')
      }</span>`
    }).join('<br><br>')}
  </div>
{/if}

<script>
import * as kv from 'idb-keyval'
import {apply_repls, split} from 'components/src/util.js'
// https://pptr.dev/api/puppeteer.page.setcontent
// word {transition: text-shadow 0.1s ease;} word.highlighted {text-shadow: 0 0 5px #FF0000;}
 
let times = []
let current_s_i = 0
let last_s_i = 0
let current_w_i = 0
let cur_time = 0
let sentences = []
let style_el

;(async () => {
    times = await kv.get('times') || []
    sentences = await kv.get('sentences') || []
    if (!times.length)
        set_times(await fetch('https://files.nuqayah.com/timing/book.csv').then(r => r.text()))
    if (!sentences.length)
        set_sentences(await fetch('https://files.nuqayah.com/timing/1.txt').then(r => r.text()))
    window.sentences = sentences
    console.log({times})
})()

function time_updated() {
    if (!times.length) return
    cur_time = audio_el.currentTime

    current_s_i = 0
    while (current_s_i < times.length && times[current_s_i][1] < cur_time)
        current_s_i++
    if (current_s_i !== last_s_i)
        document.querySelector(`[data-sentence-id="${current_s_i + 1}"]`)?.scrollIntoView({behavior: 'smooth', block: 'nearest'})

    current_w_i = 0
    while (current_w_i < times[current_s_i][2].length && times[current_s_i][2][current_w_i][1] < cur_time)
        current_w_i++

    console.log(cur_time, current_s_i, current_w_i)
    if (style_el)
        style_el.innerText = apply_repls(style_el.innerText, [
            [/data-sentence-id="\d+"/, `data-sentence-id="${current_s_i + 1}"`],
            [/data-word-id="[\d-]+"/, `data-word-id="${current_s_i + 1}-${current_w_i + 1}"`],
        ])
    last_s_i = current_s_i
}
function prepare_sentences(s) {
    return apply_repls(s, [
        [/\n+/g, ' '],
        ['.', '|'],
        [/[^\p{sc=Arabic}\p{N} |]/gu, ''],
    ]).split('|').map(s => s.split(' '))
}

async function set_sentences(e) {
    let val = ''
    if (e.preventDefault) {
        e.preventDefault()
        val = e.clipboardData.getData('text/plain').trim()
    }
    else
        val = e.trim()
    sentences = prepare_sentences(val)
    await kv.set('sentences', sentences)
    console.log(sentences)
}
async function set_times(e) {
    let val = ''
    if (e.preventDefault) {
        e.preventDefault()
        val = e.clipboardData.getData('text/plain').trim()
    }
    else
        val = e.trim()
    times = val.split('\n').map(l => {
        const parts = split(l, ',', 6).slice(3)
        parts[0] = +parts[0] / 1000
        parts[1] = +parts[1] / 1000
        parts[2] = parts[2].split(',').map(w => w.split(':').slice(1).map(i => +i / 1000))
        return parts
    })
    window.times = times
    await kv.set('times', times)
}

function text_click(e) {
    // if (e.target.tagName !== 'SPAN') return
    // const [s_i, w_i] = e.target.dataset.wordId.split('-').map(i => +i - 1)
    // audio_el.currentTime = times[s_i][2][w_i][0]
    // audio_el.play()
}
</script>
