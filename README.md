# The Cubic Seal - Geometrický Konsenzus

Tato větev obsahuje technickou dokumentaci a vizualizaci algoritmu **The Cubic Seal**.
<br>This branch contains technical documentation and a visualization of The Cubic Seal algorithm.</br>

### 🔑 Klíčové vlastnosti / Key Features:

* **Sequential Labyrinth & Bitrate Corridor:** *(CZ)* 10 000 sekvenčně závislých skoků na disku. Adresa každého skoku kryptograficky závisí na datech z předchozího. V kombinaci s limitem čtení (250–3 000 MB/s) to absolutně znemožňuje softwarovou emulaci latence v RAM.
 <br> *(EN)* 10,000 sequentially dependent disk jumps. Each jump's address cryptographically depends on the previous read. Combined with a read limit (250–3,000 MB/s), this absolutely prevents software emulation of latency in RAM.</br>

* **Proof-of-Physical-Read (PoPR):** *(CZ)* Blesková síťová validace. Vítěz prokazuje fyzické čtení z disku pomocí 10 deterministických "drobků" (Breadcrumbs). Ověření ostatními uzly trvá <10 ms.
   <br>*(EN)* Lightning-fast network validation. The winner proves physical disk reads using 10 deterministic "Breadcrumbs". Verification by other nodes takes <10 ms.</br>

* **Dynamic Codex Curvature (Anti-Farm):** *(CZ)* Každý uzel má svůj Codex šifrovaný vlastním Hardware ID (Hardware Salt) a s každým novým blokem se aplikuje nová XOR maska (Codex Salt). Statické kopie dat pro těžební farmy jsou bezcenné.
   <br>*(EN)* Each node's Codex is encrypted with its own Hardware ID (Hardware Salt), and a new XOR mask (Codex Salt) is applied with every new block. Static data copies for mining farms are worthless.</br>

* **Adaptive Geometric Difficulty (Radius R):** *(CZ)* Deterministická úprava obtížnosti (poloměru cíle v 3D prostoru) na základě mediánu času minulých epoch. Udržuje stabilní 60s takt sítě bez nutnosti centrálních hodin.
  <br> *(EN)* Deterministic adjustment of difficulty (target radius in 3D space) based on the Median Time Past of previous epochs. Maintains a stable 60s network heartbeat without a central clock.</br>

* **The Sharded Universe (Horizontal Scaling):** *(CZ)* Pokud síť přesáhne kapacitu ~8 milionů uživatelů, 512MB Codex se nenafukuje, ale štěpí do paralelních "Dimenzí". Hardwarové nároky na Lovce (uzly) zůstávají navždy minimální.
   <br>*(EN)* If the network exceeds ~8 million users, the 512MB Codex doesn't inflate but splits into parallel "Dimensions". Hardware requirements for Hunters (nodes) remain strictly minimal forever.</br>
 
 viz https://github.com/fiserjn-ship-it/Bohemiancoin-v2/blob/doc_technical/Cubic%20Seal_diagram_EN.png?b
 <br> viz https://github.com/fiserjn-ship-it/Bohemiancoin-v2/blob/doc_technical/Cubic%20Seal_diagram_CZ.png </br>
