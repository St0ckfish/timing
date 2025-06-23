<audio {src} bind:this={audio} controls/>

<p class=mt-4>• ayah timing: <code>{times[get_ayah_id(surah, ayah) + surah - 1] || ''}</code>
<p>• ayah text: <span dir=rtl class="font-[Kitab]">{mushaf?.[get_ayah_id(surah, ayah) - 1] || ''}</span>

<fieldset class="p-1 mt-4">
  <div class=flex><label class="block flex-1">Audio base <input type=text class="block !w-full" bind:value={audio_base} on:input={set_ext}></label><label class=block>ext <input type=text bind:value={ext} class="block !w-20 !min-w-0"></label></div>
  <label class=block>Surah <input type=number bind:value={surah} class="block w-20"></label>
  <label class=block>Ayah <input type=number bind:value={ayah} class="block w-20"></label>
  <label class=block>Times <textarea class="block" on:input={e => set_times(e.target.value)} on:paste={paste}/></label>
  <button class="btn !py-1" use:filedrop on:filedrop={e => on_select(e.detail.files.accepted[0])}>Select <icon id=open></button>
  <button class=btn on:click={copy_times}><icon id=copy></button>
</fieldset>

<script>
import {read_file} from 'components/src/util.js'
import {get_ayah_id} from '~/util/mushaf.js'
import {filedrop} from 'filedrop-svelte'

const pad = i => i.toString().padStart(3, '0')
async function on_select(file) {
    set_times((await read_file(file)).trim())
}

let mushaf
fetch('https://fonts.nuqayah.com/quran.txt').then(r => r.text()).then(text => {
    mushaf = text.split('\n').map(l => l.split(',')[0])
})
let audio
let audio_base = 'https://muqri.com/audio-v2/alhusari-muallim/mp3/'
let times = []
let surah = 1
let ayah = 1
let ext = 'mp3'
const exts = ['mp3', 'wav', 'opus', 'caf']

function get_time(e) {return 1}
function set_ext() {
    const ext1 = audio_base.split('/').at(-2)
    if (exts.includes(ext1) && ext !== ext1)
        ext = ext1
}
function set_times(val) {
    times = val.split('\n').slice(1).map(l => l.split(',', 6))
    window.t = times
}
function copy_times() {
    // Sura,Ayah,IndexStart,IndexEnd,TimeStart,TimeEnd
    let times_filtered = times.filter(t => t[1] !== '0')
    times_filtered = times_filtered.map(t => [...(times_filtered.length === 6236 ? [] : t.slice(0, 2)), ...t.slice(4, 6)].map(i => +i))
    // TODO: avg times
    const text = JSON.stringify(times_filtered).replaceAll('],[', '],\n[')
    navigator.clipboard.writeText(text)
}

$: if (audio) audio.currentTime = +(times[get_ayah_id(surah, ayah) + surah - 1]?.[4] || 0) / 1000
$: src = audio_base + pad(surah) + '.' + ext
function paste(e) {
    set_times(e.clipboardData.getData('text/plain').trim())
    e.preventDefault() // avoid forcing the browser to render the text as it can be large and thus hang
}
</script>
