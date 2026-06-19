# Nauči Dizajn — Giveaway „Lični brend" forma

Multi-step prijavna forma za giveaway 3-mesečnog mentorskog programa (lični brend + dobijanje klijenata u dizajnu). Isti dizajn/stil kao prethodni giveaway — promenjena su samo pitanja.

> **Referral sistem dolazi u sledećoj fazi.** Forma je trenutno samostalna (capture `?r=CODE` koda postoji u payload-u, ali thank-you referral ekran još nije napravljen).

## Repo layout

```
/
├── README.md                       ← ovaj fajl
├── master.html                     ← source-of-truth forme (CSS + JS + HTML u jednom). RADI U OVOM FAJLU.
├── naucidizajn-giveaway.css        ← ekstrakt iz master.html (deploy artifact)
├── naucidizajn-giveaway.js         ← ekstrakt iz master.html (deploy artifact)
├── embed.html                      ← Webflow embed za LANDING (paste-uj ceo fajl u Embed Code element)
├── thankyou.html                   ← standalone thank-you stranica (za lokalni test)
├── webflow-thankyou-embed.html     ← Webflow embed za THANK-YOU (paste-uj ceo fajl u Embed Code element)
└── index.html                      ← GitHub Pages test wrapper za formu
```

## Webflow embed — šta gde ide

- **Landing stranica** → paste-uj ceo [`embed.html`](embed.html). Povlači CSS/JS sa GitHub Pages
  (forma je ~74KB, prevelika za Webflow-ov 50K inline limit) — **GitHub Pages mora biti uključen**.
- **Thank-you stranica** (`/giveaway-jun-2026-uspesna-prijava`) → paste-uj ceo
  [`webflow-thankyou-embed.html`](webflow-thankyou-embed.html). Self-contained (~13KB, ne zavisi od GitHub-a).

## Forma — pitanja po redu

1. Welcome screen sa „Prijavi se" CTA
2. **Q1** Ime i prezime (text — traži ime + prezime)
3. **Q2** Gde si trenutno u dizajnu? (4 opcije)
4. **Q3** Koju vrstu dizajna trenutno radiš najviše? (web / UI-UX / brand)
5. **Q4** Koliko si zaradio/la od dizajna u poslednja 3 meseca? (5 raspona)
6. **Q5** Link do portfolija (text — „Nemam" prolazi)
7. **Q6** Šta ti je najveći problem sa klijentima trenutno? (textarea)
8. **Q7** Šta bi za tebe bio OGROMAN uspeh za 3 meseca? (textarea)
9. **Q8** Koliko sati dnevno možeš da uložiš? (<1h / 1-2h / 2-4h / 4h+)
10. **Q9** Delimična stipendija (veliki popust)? (Da / Ne)
11. **Q10** Možeš li da kreneš krajem jula i radiš posvećeno 3 meseca? (Mogu / Ne mogu)
12. **Q11** Email adresa (email validacija)
13. **Q12** Telefon — WhatsApp / Viber (per-country picker, IP geo default)
14. **Q13** Saglasnost za kontakt (DA / NE)
15. **Q14** „DUPLIRAJ SVOJU ŠANSU" — završni ekran + „POŠALJI SVOJU PRIJAVU" dugme → submit
16. Loading screen → (prod) redirect na `/giveaway-jun-2026-uspesna-prijava`

## Konfiguracija (već postavljeno)

U `master.html` → `ND_CONFIG`:

- **`webhookUrl`** = `https://hook.eu2.make.com/dlstkyxp1k8vf20lfy6xjmhjordumax3` ✅
- **`thankYouUrl`** = `https://naucidizajn.com/giveaway-jun-2026-uspesna-prijava` ✅
- **`formId`** = `naucidizajn-giveaway-licni-brend` — razlikuje ovaj giveaway od ostalih na **istom** Make webhook-u. Ne menjaj bez razloga.

### Još otvoreno

- **`FORM.welcome`** (master.html) — draft copy hero ekrana (naslov + 2 paragrafa). Zameni finalnim tekstom kad bude spreman → re-ekstrahuj css/js.

## Referral sistem (AKTIVAN)

Thank-you kartica generiše pravi referral link preko **namenskog** Vercel projekta (odvojen od starog webinar projekta, izvor je u [`/referral`](referral)):

- **Vercel projekat:** `giveaway-licni-brend-referral` (naucidizajn nalog), deploy iz `/referral`
- **API:** `https://giveaway-licni-brend-referral.vercel.app/api/signup` (Supabase-only mod — bez Kit-a, jer `thankyou.html` ne šalje `tagId`)
- **Dashboard:** `https://giveaway-licni-brend-referral.vercel.app/?t=<token>` (link „svojoj platformi" u kartici)
- **Supabase:** `bfnutgejcpqxghyslxur.supabase.co` (isti projekat kao stari; tabela `signups` je deljena — nema kolone za kampanju, pa se pobednici filtriraju po `created_at`)

Tok: forma hvata `?r=CODE` (`captureRef` u master.html, bez landing-head snippet-a) → na submit stash-uje `{email,name,ref}` u `sessionStorage.nd_signup` → thank-you čita stash, POST-uje na `/api/signup` sa `webinarUrl = CONFIG.landingUrl` → API upiše u Supabase i vrati `{ shareUrl, dashboardUrl }`.

Konfiguracija u `thankyou.html` → `CONFIG`: `REFERRAL_ENABLED: true`, `apiUrl`, `landingUrl` (baza `?r=` linka — **mora biti URL landing stranice**). Pregled izgleda bez API poziva: `thankyou.html?demo=1`.

**Env varijable** (Vercel projekat `giveaway-licni-brend-referral`): `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `DASHBOARD_BASE_URL`.
**Redeploy** posle izmene u `/referral`: `cd referral && vercel link --project giveaway-licni-brend-referral --scope naucidizajns-projects && vercel deploy --prod` (`.vercel/` je u .gitignore, pa se prvi put linkuje).

## Webhook payload

POST na Make webhook. Keys u `answers` objektu su **puna pitanja**:

```json
{
  "form_id": "naucidizajn-giveaway-licni-brend",
  "session_id": "nd_gv_xxx_xxx",
  "completed_full": true,
  "contact": {
    "first_name": "Marko", "last_name": "Marković", "name": "Marko Marković",
    "email": "marko@test.com",
    "phone": "601112233", "phone_country": "RS", "phone_dial": "+381", "phone_full": "+381601112233"
  },
  "opt_in": true,
  "opt_in_raw": "DA",
  "answers": {
    "Gde si trenutno u dizajnu?": "…",
    "Koju vrstu dizajna trenutno radiš najviše?": "…",
    "Koliko si zaradio/la od dizajna u poslednja 3 meseca?": "…",
    "Link do portfolija (sajt, Behance, Dribbble, Google Drive ili Instagram)": "…",
    "Šta ti je najveći problem sa klijentima trenutno?": "…",
    "Ako bismo zajedno radili naredna 3 meseca, šta bi za tebe bio OGROMAN uspeh?": "…",
    "Koliko realno sati dnevno možeš da uložiš u dobijanje klijenata u naredna 3 meseca?": "…",
    "Ako ne osvojiš full stipendiju, da li bi te zanimalo da dobiješ delimičnu stipendiju (veliki popust) za isti program?": "…",
    "Ovaj mentorski program kreće krajem jula. Da li možeš da kreneš tada i uz moju pomoć radiš posvećeno sledeća 3 meseca?": "…",
    "Slažem se da me Nauči Dizajn kontaktira u vezi giveaway-a i posebnih ponuda": "DA"
  },
  "referral": { "referred_by_code": "", "source_url": "..." },
  "utm": { "utm_source": "..." }
}
```

Kontakt (ime/email/telefon) ide u `contact`, saglasnost u `opt_in` (+ duplirana u `answers`). Ostala kvalifikaciona pitanja idu u `answers` po punom tekstu pitanja.

## Kako menjati formu

Source-of-truth je **`master.html`** (CSS + JS + HTML u jednom). Posle izmena re-ekstrahuj artifact fajlove:

```bash
node -e '
const fs=require("fs"), s=fs.readFileSync("master.html","utf8");
const cut=(a,b)=>{const i=s.indexOf(a),j=s.indexOf(b,i+a.length);return s.slice(i+a.length,j);};
fs.writeFileSync("naucidizajn-giveaway.css", cut("<style>","</style>").replace(/^[\r\n]+/,"").replace(/\s+$/,"")+"\n");
fs.writeFileSync("naucidizajn-giveaway.js",  cut("<script>","</script>").replace(/^[\r\n]+/,"").replace(/\s+$/,"")+"\n");
'
```

Posle ekstrakcije **povećaj `?v=N`** u `embed.html` (i u Webflow Embed elementu) da forsiraš cache refresh.

## TEST vs PROD

TEST env detection (u JS-u): `host === 'localhost' || '127.0.0.1' || *.github.io || file://`.

U TEST env-u: webhook se NE šalje (samo `console.log`), redirect je isključen, žuti banner na vrhu.
PROD (`naucidizajn.com`): sve radi normalno.

## Lokalni dev

```bash
npx serve . -l 4630
# Otvori http://localhost:4630/master.html  (ili /index.html za artifact verziju)
```

## Deploy

1. Push na GitHub repo `naucidizajn/giveaway-licni-brend` sa fajlovima u root-u.
2. Settings → Pages → Source: `main` / `/ (root)`.
3. (Opciono) zameni `FORM.welcome` finalnim copy-jem → re-ekstrahuj css/js → push.
4. Webflow landing: dodaj Embed Code element sa sadržajem iz `embed.html` (povećaj `?v=N` na svaku izmenu).
5. Webflow stranica `/giveaway-jun-2026-uspesna-prijava`: dodaj Embed Code element sa sadržajem iz `thankyou.html` — kopiraj blok između `START: Webflow Embed` i `END: paste blok`.
