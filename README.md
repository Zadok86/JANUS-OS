# JANUS-OS
https://chatgpt.com/share/6881b7bb-a484-8010-ae55-27d00ff12cdf
{
  "event": "Assassination of Julius Caesar",
  "date": "March 15, 44 BC",
  "cause": "Stabbed 23 times by senators including Brutus and Cassius",
  "political_consequences": [
    "Collapse of the Roman Republic",
    "Civil War",
    "Rise of Octavian (Augustus) and Imperial Rome"
  ],
  "reliability_score": 92,
  "model_agreement": "High",
  "sources": [
    "Plutarch - Life of Caesar",
    "Suetonius - The Twelve Caesars"
  ],
  "flagged_models": [
    {
      "name": "Mistral",
      "reason": "Factual inaccuracy (claimed Caesar was poisoned)"
    }
  ]
}
from scapy.all import sniff
import pretty_midi
from riffusion.cli import generate
import datetime

# --- 1. Packet Capture ---
packets = []

def packet_callback(packet):
    try:
        size = len(packet)
        src = packet[0][1].src if packet.haslayer(1) else '0.0.0.0'
        proto = packet.proto if hasattr(packet, 'proto') else 0
        packets.append((size, proto, src))
    except Exception as e:
        print(f"Error: {e}")

sniff(prn=packet_callback, count=500)  # Capture 500 packets for test

# --- 2. Map Packets to MIDI ---
midi = pretty_midi.PrettyMIDI()
instrument = pretty_midi.Instrument(program=0)
time = 0

for size, proto, src in packets:
    pitch = min(max(int(size / 5), 21), 108)  # Map packet size to piano range
    duration = 0.2
    note = pretty_midi.Note(velocity=100, pitch=pitch, start=time, end=time + duration)
    instrument.notes.append(note)
    time += 0.25

midi.instruments.append(instrument)
filename = f"janus_{datetime.datetime.now().strftime('%Y%m%d_%H%M%S')}.mid"
midi.write(filename)

# --- 3. Generate Audio with Riffusion ---
audio_file = generate(
    prompt="cybernetic symphony, dark ambient, generative melody",
    midi=filename,
    output=f"{filename}.wav"
)
print(f"Generated Audio: {audio_file}")
