# HA_micro_wake_word
Home Assistant Voice Preview Edition (VPE) zpracovává probouzecí fráze (Wake Words) přímo na svém čipu (on-device) pomocí technologie microWakeWord. 

Zařízení Home Assistant Voice Preview Edition (VPE) zpracovává probouzecí fráze (Wake Words) přímo na svém čipu (on-device) pomocí technologie microWakeWord. Z výroby má v sobě nahrané tři standardní fráze: „Okay Nabu“, „Hey Jarvis“ a „Hey Mycroft“.
Pokud chcete zcela vlastní frázi (např. „Ahoj dome“), máte dvě možnosti, jak toho docílit.

Možnost A: Jednodušší cesta přes server (openWakeWord)

Místo abyste nutili slabý čip v zařízení složitě rozpoznávat nová slova, necháte ho pouze posílat zvuk na váš Home Assistant server, kde bude běžet doplněk openWakeWord.
1. Vytvoření fráze: Přejděte na bezplatný nástroj pro kompilaci microWakeWord / openWakeWord. Zadejte svou frázi, nechte ji vygenerovat umělou inteligencí (využívá se syntéza řeči Piper) a stáhněte si výsledný model ve formátu .tflite.
2. Nahrání do HA: Přes doplněk Samba share nebo File Editor nahrajte stažený soubor .tflite do složky /share/openwakeword/ ve vašem Home Assistantu.
3. Aktivace v Hlasovém asistentovi:
	• Přejděte do Nastavení -> Hlasoví asistenti.
	• Vyberte svého asistenta, v sekci Probouzecí slovo zvolte openWakeWord a následně vyberte ze seznamu svou nahranou frázi.
4. Přepnutí zařízení: Přejděte do Nastavení -> Zařízení a služby -> ESPHome -> klikněte na vaše Voice PE. V sekci konfigurace změňte detekci z interní na asistenta z Home Assistantu.
Nevýhoda: Zařízení ztratí možnost okamžité lokální reakce (LED kroužek blikne až poté, co slovo potvrdí server).

Možnost B: Pokročilá cesta přímo na zařízení (microWakeWord)

Pokud jste již převzali kontrolu nad zařízením v ESPHome Builderu (podle předchozích kroků), můžete vlastní model typu .tflite (vygenerovaný a stažený z microwakeword.com) integrovat přímo do firmwaru zařízení.
Vzhledem k tomu, že balíček od Nabu Casa obsahuje tovární fráze, musíte je v YAML souboru nejprve odebrat a následně podstrčit odkaz na váš vlastní model uložený na internetu (např. na vašem GitHubu).
Upravte svůj YAML v ESPHome Device Builderu následovně:
yaml
esphome:
  name: home-assistant-voice-0a9e2d
  friendly_name: Voice Preview Edition

substitutions:
  name: home-assistant-voice-0a9e2d
  friendly_name: Voice Preview Edition

packages:
  nabu_casa.voice_pe: github://esphome/home-assistant-voice-pe/home-assistant-voice.yaml@main

# ODEBRÁNÍ TOVÁRNÍCH SLOV A PŘIDÁNÍ VLASTNÍHO
micro_wake_word:
  # Odebereme výchozí slova, která nechcete používat, aby nezabírala paměť RAM
  models:
    - model: hey_jarvis
      id: !remove hey_jarvis_model
    - model: hey_mycroft
      id: !remove hey_mycroft_model
      
    # Přidání vašeho vlastního zkompilovaného modelu (včetně konfiguračního JSONu)
    - model:
        url: https://githubusercontent.com
        type: http
      id: moje_vlastni_fraze_model

# Nastavení, aby asistent po startu poslouchal primárně vaše nové slovo
voice_assistant:
  wake_word: moje_vlastni_fraze_model

light:
  - id: !extend led_ring_light
    channel_colors: GRB
Při použití kódu buďte obezřetní.

Jak musí vypadat soubor .json na vašem úložišti?

Aby ESPHome věděl, jak model zpracovat, musíte na internet (např. GitHub) nahrát jak soubor .tflite, tak .json soubor, který na něj odkazuje. Obsah .json souboru vypadá takto:
json
{
  "type": "micro",
  "wake_word": "Ahoj dome",
  "version": 2,
  "model_path": "https://githubusercontent.com",
  "micro": {
    "probability_cutoff": 0.5,
    "sliding_window_size": 10
  }
}
Při použití kódu buďte obezřetní.
Po uložení kódu v ESPHome Device Builderu a kliknutí na Install se vaše nová fráze napevno zkompiluje do zařízení. Detekce pak bude stoprocentně lokální, okamžitá a nezávislá na rychlosti vaší Wi-Fi sítě.
