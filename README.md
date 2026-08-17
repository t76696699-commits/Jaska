// ════════════════════════════════════════════════════════════════════
// 3-BOSQICH: Vanilla JS frontend - Flask orqali serverlanadi
// ════════════════════════════════════════════════════════════════════

// ─────────────────────────────────────────────────────────────────────
// 1) app/static/app.js - ma'lumot olish va chizish
// ─────────────────────────────────────────────────────────────────────

async function xarajatlarniYuklash() {
  const javob = await fetch('/api/expenses');
  const xarajatlar = await javob.json();
  royxatniChizish(xarajatlar);
}

// ─────────────────────────────────────────────────────────────────────
// 2) TO'G'RI: let bilan sikl - har bir tugma o'z x.id'siga ishora qiladi
// ─────────────────────────────────────────────────────────────────────

function royxatniChizish(xarajatlar) {
  const royxat = document.getElementById('xarajatlar-royxati');
  royxat.innerHTML = '';

  for (let i = 0; i < xarajatlar.length; i++) {
    const x = xarajatlar[i];
    const li = document.createElement('li');
    li.textContent = `${x.tavsif}: ${x.summa} so'm `;

    const ochirishTugmasi = document.createElement('button');
    ochirishTugmasi.textContent = "O'chirish";
    ochirishTugmasi.addEventListener('click', () => {
      xarajatniOchirish(x.id);
    });

    li.appendChild(ochirishTugmasi);
    royxat.appendChild(li);
  }
}

async function xarajatniOchirish(id) {
  await fetch(`/api/expenses/${id}`, { method: 'DELETE' });
  xarajatlarniYuklash();
}

xarajatlarniYuklash();

// ─────────────────────────────────────────────────────────────────────
// 3) Ataylab xato - 'var' bilan sikl (izohda)
// ─────────────────────────────────────────────────────────────────────

// function royxatniChizishXato(xarajatlar) {
//   for (var i = 0; i < xarajatlar.length; i++) {   // var ishlatilgan!
//     const tugma = document.createElement('button');
//     tugma.addEventListener('click', () => {
//       console.log(i);   // HAR DOIM oxirgi qiymatni chiqaradi!
//     });
//   }
// }
