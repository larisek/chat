# Šifra — návod

## Instalace na telefon
Appka je čistě statické soubory (HTML/JS) — potřebuje jen web prostor, žádný backend/databázi.

1. Nahrajte celou složku (`index.html`, `manifest.json`, `sw.js`, `icon-192.png`, `icon-512.png`) na libovolný **https** hosting — nejjednodušší je zdarma:
   - GitHub Pages (nahrát složku do repa, zapnout Pages)
   - Netlify Drop (netlify.com/drop) — přetáhnete složku a je hotovo
   - Cloudflare Pages
2. Otevřete výslednou URL na telefonu v Chromu/Safari.
3. Menu prohlížeče → **„Přidat na plochu"** / „Add to Home Screen" — appka se nainstaluje jako běžná ikonka.
4. Stejný postup zopakujte na druhém telefonu.

**Proč https hosting, ne jen otevřít soubor přímo:** mobilní prohlížeče povolují kameru (na skenování QR) a service worker (offline provoz) jen na zabezpečeném originu. Bez hostingu appka pořád funguje, ale pro párování musíte použít záložku „Vložit text" místo kamery.

## Párování dvou zařízení (proběhne jen jednou)
1. Na zařízení A: **+** → „Vytvořit nové párování" → zobrazí se tajná fráze a QR kód.
2. Frázi řekněte nahlas osobě u zařízení B (nebo pošlete jiným kanálem, ideálně ne stejným jako QR kód).
3. Na zařízení B: **+** → „Mám kód od druhého zařízení" → zadejte frázi → naskenujte QR ze zařízení A (nebo vložte zkopírovaný text).
4. Zařízení B zobrazí odpovědní QR kód → naskenujte ho zpět na zařízení A.
5. Hotovo — zařízení jsou natrvalo spárována a mohou si posílat šifrované zprávy.

## Jak to funguje technicky (stručně)
- **Přenos zpráv**: přímé WebRTC P2P spojení mezi telefony (žádný server zprávy nikdy nevidí).
- **Šifrování párování**: nabídka/odpověď spojení je zašifrovaná AES-256-GCM klíčem odvozeným z vaší tajné fráze (PBKDF2, 150 000 iterací) — bez správné fráze nelze párovací kód použít.
- **Šifrování zpráv**: po spárování se pro každou zprávu používá klíč odvozený ECDH (P-256) z dlouhodobých klíčů obou zařízení — end-to-end, jen tato dvě zařízení ho můžou spočítat.
- **Ověření**: v chatu je ikona 🔑 — zobrazí "otisk" spojení, který si strany mohou nahlas porovnat (jako u Signalu), aby si potvrdily, že spojení nikdo nepodvrhl.

## Důležité technické omezení (poctivě říct dopředu)
Skutečné P2P bez ANY serveru má jeden nevyhnutelný limit: aby se dvě zařízení na internetu vůbec našla přes NAT, musí si **jednou za session** vyměnit spojovací údaje. To dělá tato appka ručně (QR kód / zkopírovaný text) — proto po zavření appky/výpadku sítě musíte znovu "Připojit" (v chatu), což ale už nevyžaduje frázi — jen krátkou výměnu kódu, protože zařízení už si navzájem důvěřují.

Appka používá veřejné STUN servery (Google) jen pro zjištění vaší veřejné IP/portu (standardní součást WebRTC) — ty nikdy neuvidí obsah zpráv ani šifrovací klíče.

## Data
Zprávy a klíče se ukládají pouze lokálně v prohlížeči (IndexedDB) na daném zařízení. Smazání appky/dat prohlížeče = nenávratná ztráta historie a klíčů.
