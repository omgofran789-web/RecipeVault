// ===== بيانات تجريبية للوصفات (تخزين محلي) =====
let recipes = [];

// تحميل الوصفات من localStorage إن وجدت، وإلا ضع بيانات افتراضية
function loadRecipes() {
  const stored = localStorage.getItem('recipes');
  if (stored) {
    recipes = JSON.parse(stored);
  } else {
    recipes = [
      {
        id: 1,
        name: 'كبسة دجاج',
        category: 'رئيسية',
        ingredients: 'دجاج، أرز بسمتي، بهارات كبسة، بصلة، ثوم، زيت',
        instructions: '1. اقلي البصل والثوم، أضيفي الدجاج وقلبيه. 2. أضيفي الأرز والماء والبهارات، اتركيه حتى ينضج.'
      },
      {
        id: 2,
        name: 'سلطة الفواكه',
        category: 'مقبلات',
        ingredients: 'تفاح، موز، برتقال، عنب، عسل، ليمون',
        instructions: 'قطعي الفواكه واخلطيها مع العسل وعصير الليمون، قدميها باردة.'
      }
    ];
    saveRecipes();
  }
  return recipes;
}

function saveRecipes() {
  localStorage.setItem('recipes', JSON.stringify(recipes));
}

// ===== عرض الوصفات في لوحة التحكم =====
function renderRecipes() {
  const grid = document.getElementById('recipeGrid');
  if (!grid) return;
  const all = loadRecipes();
  grid.innerHTML = '';
  if (all.length === 0) {
    grid.innerHTML = '<p style="text-align:center;color:#888;">لا توجد وصفات حالياً، أضيفي وصفة جديدة!</p>';
    return;
  }
  all.forEach(recipe => {
    const card = document.createElement('div');
    card.className = 'recipe-card';
    card.innerHTML = `
      <h3>${recipe.name}</h3>
      <span class="category">${recipe.category}</span>
      <div class="ingredients"><strong>المكونات:</strong> ${recipe.ingredients}</div>
      <div class="instructions"><strong>التحضير:</strong> ${recipe.instructions}</div>
    `;
    grid.appendChild(card);
  });
}

// ===== تسجيل الدخول =====
function setupLogin() {
  const form = document.getElementById('loginForm');
  if (!form) return;
  form.addEventListener('submit', function(e) {
    e.preventDefault();
    const username = document.getElementById('username').value.trim();
    const password = document.getElementById('password').value.trim();
    const errorDiv = document.getElementById('loginError');
    if (username === 'admin' && password === '123') {
      sessionStorage.setItem('loggedIn', 'true');
      window.location.href = 'dashboard.html';
    } else {
      errorDiv.textContent = '❌ اسم المستخدم أو كلمة المرور غير صحيحة!';
    }
  });
}

// ===== إضافة وصفة =====
function setupAddRecipe() {
  const form = document.getElementById('addRecipeForm');
  if (!form) return;
  form.addEventListener('submit', function(e) {
    e.preventDefault();
    const name = document.getElementById('recipeName').value.trim();
    const category = document.getElementById('recipeCategory').value.trim();
    const ingredients = document.getElementById('recipeIngredients').value.trim();
    const instructions = document.getElementById('recipeInstructions').value.trim();
    const msg = document.getElementById('addMessage');
    if (!name || !category || !ingredients || !instructions) {
      msg.textContent = '⚠️ جميع الحقول مطلوبة!';
      msg.style.color = '#c0392b';
      return;
    }
    // حفظ
    const all = loadRecipes();
    const newId = all.length ? Math.max(...all.map(r => r.id)) + 1 : 1;
    all.push({ id: newId, name, category, ingredients, instructions });
    saveRecipes();
    msg.textContent = '✅ تمت إضافة الوصفة بنجاح!';
    msg.style.color = '#27ae60';
    form.reset();
    // تحديث الـ Dashboard إذا كان مفتوحاً (سيتم تحديثه عند الانتقال)
  });
}

// ===== نموذج التواصل =====
function setupContact() {
  const form = document.getElementById('contactForm');
  if (!form) return;
  form.addEventListener('submit', function(e) {
    e.preventDefault();
    const name = document.getElementById('contactName').value.trim();
    const email = document.getElementById('contactEmail').value.trim();
    const message = document.getElementById('contactMessage').value.trim();
    const status = document.getElementById('contactMessageStatus');
    if (!name || !email || !message) {
      status.textContent = '⚠️ جميع الحقول مطلوبة!';
      status.style.color = '#c0392b';
      return;
    }
    // محاكاة الإرسال
    status.textContent = `✅ شكراً ${name}، تم استلام رسالتك! سنرد على ${email} قريباً.`;
    status.style.color = '#27ae60';
    form.reset();
  });
}

// ===== تسجيل الخروج =====
function setupLogout() {
  const btn = document.getElementById('logoutBtn');
  if (btn) {
    btn.addEventListener('click', function(e) {
      e.preventDefault();
      sessionStorage.removeItem('loggedIn');
      window.location.href = 'index.html';
    });
  }
}

// ===== حماية الصفحات (تأكد من تسجيل الدخول) =====
function checkAuth() {
  // الصفحات المحمية: dashboard, add-recipe, contact
  const protectedPages = ['dashboard.html', 'add-recipe.html', 'contact.html'];
  const current = window.location.pathname.split('/').pop();
  if (protectedPages.includes(current)) {
    const logged = sessionStorage.getItem('loggedIn');
    if (logged !== 'true') {
      window.location.href = 'index.html';
    }
  }
  // إذا كنت في login ومسجل دخول، اذهب للـ dashboard
  if (current === 'index.html' || current === '') {
    if (sessionStorage.getItem('loggedIn') === 'true') {
      window.location.href = 'dashboard.html';
    }
  }
}

// ===== التشغيل عند تحميل الصفحة =====
document.addEventListener('DOMContentLoaded', function() {
  checkAuth();
  setupLogin();
  setupAddRecipe();
  setupContact();
  setupLogout();
  renderRecipes(); // لعرض الوصفات في dashboard
});
