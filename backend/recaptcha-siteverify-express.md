# reCAPTCHA siteverify — back Express

Stack: Node/Express, fetch natif.
Usage: vérification serveur du token reCAPTCHA (partie back du flux décrit dans [recaptcha-v2-v3.md](recaptcha-v2-v3.md)).

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

  res.json({ ok: true })
})
```

Notes : v2 = retirer action/score. Échec réseau vers Google = échouer fermé. Logger le score v3 pour ajuster le seuil.

## État
**vivant** — partie back du flux reCAPTCHA complet (voir recaptcha-v2-v3.md).
