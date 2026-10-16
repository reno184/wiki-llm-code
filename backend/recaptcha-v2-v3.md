# Google reCAPTCHA — principes + front (Vue 3) + back (Node/Express)

Stack: Vue 3 Composition API, Node/Express, fetch natif.
Usage: protection formulaires (contact, inscription) contre bots/spam.

## Principes (à relire avant d'implémenter)

1. **Le token n'est jamais validé côté front.** Il n'a aucune valeur de sécurité tant que le back ne l'a pas vérifié auprès de Google.
2. **Flux** : front obtient un token → l'envoie avec le formulaire → back appelle https://www.google.com/recaptcha/api/siteverify avec le token **et la clé secrète** → Google répond success (v3 : + score et action).
3. **La clé secrète ne vit jamais dans le front.** Variable d'environnement côté serveur uniquement.
4. **Le token expire vite (~2 min) et est à usage unique** : vérification synchrone à la soumission.
5. **v2 (case à cocher)** : binaire, UX intrusive, formulaires publics à fort spam. **v3 (score invisible)** : score 0.0-1.0 sans interaction ; seuil (ex. >= 0.5) = décision métier côté back.
6. Combiner toujours avec une validation serveur des champs : reCAPTCHA protège des bots, pas des attaques applicatives.

## Front (Vue 3, v3)

```vue
<script setup>
import { ref, onMounted } from 'vue'

const SITE_KEY = import.meta.env.VITE_RECAPTCHA_SITE_KEY // public, ok dans le front
const emit = defineEmits(['submitted'])
const sending = ref(false)
const error = ref('')
let ready = false

function loadRecaptcha() {
  return new Promise((resolve) => {
    if (window.grecaptcha) return resolve()
    const s = document.createElement('script')
    s.src = 'https://www.google.com/recaptcha/api.js?render=' + SITE_KEY
    s.onload = resolve
    document.head.appendChild(s)
  })
}

onMounted(async () => {
  await loadRecaptcha()
  window.grecaptcha.ready(() => { ready = true })
})

async function getToken(action) {
  if (!ready) throw new Error('reCAPTCHA pas chargé')
  return window.grecaptcha.execute(SITE_KEY, { action })
}

async function submit(form) {
  sending.value = true; error.value = ''
  try {
    const token = await getToken('submit_contact')
    const res = await fetch('/api/contact', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ ...form, recaptchaToken: token })
    })
    if (!res.ok) throw new Error('Erreur serveur')
    emit('submitted')
  } catch (e) {
    error.value = 'Validation anti-spam échouée, réessayez.'
  } finally { sending.value = false }
}
</script>
```

Notes front : v2 = `grecaptcha.render` + lire `g-recaptcha-response` au lieu d'`execute`. Conserver l'`action` : le back la vérifie aussi en v3.

## Back (Node/Express)

```js
const SECRET = process.env.RECAPTCHA_SECRET_KEY // jamais dans le front
const MIN_SCORE = 0.5
const EXPECTED_ACTION = 'submit_contact'

async function verifyRecaptcha(token) {
  const params = new URLSearchParams({ secret: SECRET, response: token })
  const res = await fetch('https://www.google.com/recaptcha/api/siteverify', {
    method: 'POST', body: params
  })
  return res.json() // { success, score, action, ... }
}

router.post('/api/contact', async (req, res) => {
  const { name, email, message, recaptchaToken } = req.body
  if (!name || !email || !message || !recaptchaToken)
    return res.status(400).json({ error: 'Champs manquants' })

  const v = await verifyRecaptcha(recaptchaToken)
  // v2 : success suffit. v3 : success + action + score >= seuil
  const ok = v.success && v.action === EXPECTED_ACTION && (v.score ?? 1) >= MIN_SCORE
  if (!ok) return res.status(403).json({ error: 'Validation reCAPTCHA échouée' })

  // traitement métier (email, DB, etc.)
  res.json({ ok: true })
})
```

Notes back : v2 = retirer action/score, success suffit. Échec réseau vers Google = échouer fermé. Logger le score v3 pour ajuster le seuil.

## Clés

- `SITE_KEY` : front, public (`VITE_RECAPTCHA_SITE_KEY`). `SECRET` : serveur uniquement (`RECAPTCHA_SECRET_KEY`, jamais commitée).

## État

**vivant** — version unique pour tous les projets. Alternatives non retenues : honeypot seul (insuffisant), Cloudflare Turnstile (à réévaluer si besoin sans cookie Google).
