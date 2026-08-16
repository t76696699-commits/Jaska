# ════════════════════════════════════════════════════════════════════
# 3-BOSQICH: React frontend - Django API'ga ulanish
# ════════════════════════════════════════════════════════════════════

// ─────────────────────────────────────────────────────────────────────
// 1) frontend/src/api/topshiriqlar.js
// ─────────────────────────────────────────────────────────────────────

const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:8000';

export async function topshiriqlarniOlish() {
  const javob = await fetch(`${API_URL}/api/topshiriqlar/`);
  if (!javob.ok) throw new Error('Topshiriqlarni olishda xato');
  return await javob.json();
}

// ─────────────────────────────────────────────────────────────────────
// 2) frontend/src/components/TopshiriqRoyxati.jsx (izohda - JSX)
// ─────────────────────────────────────────────────────────────────────

// function TopshiriqRoyxati() {
//   const [royxat, setRoyxat] = useState([]);
//   const [holat, setHolat] = useState('yuklanmoqda');
//
//   useEffect(() => {
//     topshiriqlarniOlish()
//       .then((data) => { setRoyxat(data); setHolat('muvaffaqiyatli'); })
//       .catch(() => setHolat('xato'));
//   }, []);
//
//   if (holat === 'yuklanmoqda') return <p>Yuklanmoqda...</p>;
//
//   return (
//     <ul>
//       {royxat.map((t) => (
//         <li key={t.id}>{t.sarlavha} ({t.fan_nomi}) — {t.muddat_vaqti}</li>
//       ))}
//     </ul>
//   );
// }

# ─────────────────────────────────────────────────────────────────────
# 3) studymate/settings.py - django-cors-headers sozlash (Python, izohda)
# ─────────────────────────────────────────────────────────────────────

# INSTALLED_APPS = [
#     # ...
#     'corsheaders',
# ]
#
# MIDDLEWARE = [
#     'corsheaders.middleware.CorsMiddleware',      # CommonMiddleware'dan OLDIN!
#     'django.middleware.common.CommonMiddleware',
#     # ...
# ]
#
# CORS_ALLOWED_ORIGINS = [
#     'http://localhost:3000',
# ]

# ─────────────────────────────────────────────────────────────────────
# 4) Ataylab xato - middleware tartibini almashtirish (izohda)
# ─────────────────────────────────────────────────────────────────────

# MIDDLEWARE = [
#     'django.middleware.common.CommonMiddleware',   # CorsMiddleware'dan OLDIN - XATO!
#     'corsheaders.middleware.CorsMiddleware',
# ]
# ❌ CORS_ALLOWED_ORIGINS to'g'ri bo'lsa ham, tartib xato bo'lgani uchun CORS ishlamaydi
