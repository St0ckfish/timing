<audio {src} bind:this={audio} class="block w-full" controls/>

<fieldset>
<label>Audio base <input type=text class="block !w-full" bind:value={audio_base}></label>
<label>DB <input type=text class="block !w-full" bind:value={db_url}></label>
<label>Surah <input type=number bind:value={surah} class="block w-20"></label>
<label>Ayah <input type=number bind:value={ayah} class="block w-20"></label>
</fieldset>

<script>
import JSZip from 'jszip'
import sqlite3InitModule from 'sqlite-wasm-esm'


const pad = i => i.toString().padStart(3, '0')

const cors_prox = 'https://nxsmrlyibvtc.slk.is/cors/'
let audio_base = 'https://download.quranicaudio.com/quran/wadee_hammadi_al-yamani/'
let db_url = 'https://files.quran.app/hafs/databases/audio/wadee3_alyamani.zip'

let surah = 1
let ayah = 1
let audio
let sqlite3
let db

async function setup() {
    if (!sqlite3)
        sqlite3 = await sqlite3InitModule()
    db = load_db(await get_and_unzip_db(db_url))
}

const select = sql => db.exec(sql, {returnValue: 'resultRows'})
async function get_and_unzip_db(db_url) {
    const file_ref = await (await fetch(cors_prox + db_url)).blob()
    const db_name = db_url.split('/').at(-1).split('.')[0]
    return (await JSZip.loadAsync(file_ref)).file(db_name + '.db').async('uint8array')
}
function load_db(array_buffer) {
    const p = sqlite3.wasm.allocFromTypedArray(array_buffer)
    const len = array_buffer.byteLength
    const db = new sqlite3.oo1.DB()
    sqlite3.capi.sqlite3_deserialize(db.pointer, 'main', p, len, len, sqlite3.capi.SQLITE_DESERIALIZE_FREEONCLOSE)
    return db
}
function get_time(surah, ayah) {
    const val = select(`select time from timings where sura=${surah} and ayah=${ayah}`)[0]?.[0]
    return (val || 1e3) / 1000
}

$: if (audio && db)
    audio.currentTime = get_time(surah, ayah)
$: setup(db_url)
$: src = audio_base + pad(surah) + '.mp3'
</script>
