---
title: "About"
description: "Profile"
---

{{< rawhtml >}}
<div style="display: flex; justify-content: center; margin-top: 60px; font-family: 'Helvetica', 'Arial', sans-serif';">

  <div id="profile-card" style="
    max-width: 420px;
    width: 90%;
    padding: 40px 30px;
    border-radius: 20px;
    text-align: center;
    box-shadow: 0 12px 30px rgba(0,0,0,0.15);
    background-color: #ffffff;
    color: #222222;
    transition: transform 0.3s ease, background-color 0.3s ease, color 0.3s ease, box-shadow 0.3s ease;
  ">

    <!-- 프로필 이미지 -->
    <img src="https://tistory1.daumcdn.net/tistory/5900884/attach/f4ac8e81ef3f457b9099402bc80e2ebe"
         alt="profile photo"
         style="
           width: 140px;
           height: 140px;
           border-radius: 50%;
           object-fit: cover;
           margin: 0 auto 20px auto;
           display: block;
           box-shadow: 0 6px 20px rgba(0,0,0,0.25);
           transition: box-shadow 0.3s ease;
         " />

    <!-- 이름 -->
    <h1 style="font-size: 26px; margin: 0 0 5px 0;">신승환</h1>

    <!-- 생일 -->
    <p style="font-size: 13px; margin: 0 0 10px 0;">2006.05.08</p>

    <!-- 소속 -->
    <p style="font-size: 15px; margin-bottom: 15px;">
      고려대학교 인공지능사이버보안학과 25학번<br>
      AICS 25 | KOREA UNIV
    </p>

    <!-- 이메일 -->
    <p style="font-size: 14px; font-weight: 600; margin-bottom: 15px;">
      ku_ssh@korea.ac.kr
    </p>

    <!-- 소셜 이미지 버튼 -->
    <div style="display: flex; justify-content: center; gap: 15px; margin-bottom: 0;">
      <a href="https://github.com/laphael58" target="_blank" aria-label="Github" class="social-btn">
        <img src="https://cdn-icons-png.flaticon.com/512/25/25231.png" alt="Github">
      </a>
      <a href="https://www.instagram.com/tlstmdghks_/" target="_blank" aria-label="Instagram" class="social-btn">
        <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/a/a5/Instagram_icon.png/1200px-Instagram_icon.png" alt="Instagram">
      </a>
    </div>

  </div>
</div>

<style>
/* 카드 기본 스타일 */
#profile-card {
  background-color: #ffffff;
  color: #222222;
  transition: transform 0.3s ease, background-color 0.3s ease, color 0.3s ease, box-shadow 0.3s ease;
  cursor: default;
}

/* 카드 hover 팝업 효과 */
#profile-card:hover {
  transform: translateY(-8px) scale(1.02);
  box-shadow: 0 20px 40px rgba(0,0,0,0.3);
}

/* 텍스트 색상 */
#profile-card h1,
#profile-card p,
#profile-card span,
#profile-card a {
  color: #222222;
  transition: color 0.3s ease;
}

#profile-card .social-btn {
  display: block;          /* inline-block → block */
  margin: 0;
  padding: 0;
  border: none;
  text-decoration: none;
  outline: none;
}

#profile-card .social-btn img {
  display: block;          /* inline에서 block으로 변경 */
  width: 32px;
  height: 32px;
  border-radius: 50%;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  vertical-align: middle;  /* img alignment 제거 */
}

#profile-card .social-btn img:hover {
  transform: scale(1.2);
  box-shadow: 0 4px 12px rgba(0,0,0,0.25);
}

/* 다크 모드 */
@media (prefers-color-scheme: dark) {
  #profile-card {
    background-color: #222222;
    color: #ffffff;
    box-shadow: 0 12px 30px rgba(0,0,0,0.5);
  }
  #profile-card h1,
  #profile-card p,
  #profile-card span,
  #profile-card a {
    color: #ffffff;
  }
  #profile-card img {
    box-shadow: 0 6px 20px rgba(255,255,255,0.15);
  }
  #profile-card:hover {
    box-shadow: 0 20px 40px rgba(255,255,255,0.2);
  }
}

/* 라이트 모드 */
@media (prefers-color-scheme: light) {
  #profile-card {
    background-color: #ffffff;
    color: #222222;
    box-shadow: 0 12px 30px rgba(0,0,0,0.15);
  }
  #profile-card h1,
  #profile-card p,
  #profile-card span,
  #profile-card a {
    color: #222222;
  }
  #profile-card img {
    box-shadow: 0 6px 20px rgba(0,0,0,0.25);
  }
  #profile-card:hover {
    box-shadow: 0 20px 40px rgba(0,0,0,0.25);
  }
}
</style>

<script>
// PaperMod 테마 변경 감지 및 카드 색상 업데이트
function updateProfileCardTheme() {
  const card = document.getElementById('profile-card');
  if (!card) return;

  const theme = localStorage.getItem('pref-theme');

  if (theme === 'dark') {
    card.style.backgroundColor = '#222222';
    card.style.color = '#ffffff';
    card.style.boxShadow = '0 12px 30px rgba(0,0,0,0.5)';
    const textElements = card.querySelectorAll('h1, p, span, a');
    textElements.forEach(el => el.style.color = '#ffffff');
    const img = card.querySelector('img');
    if (img) img.style.boxShadow = '0 6px 20px rgba(255,255,255,0.15)';
  } else {
    card.style.backgroundColor = '#ffffff';
    card.style.color = '#222222';
    card.style.boxShadow = '0 12px 30px rgba(0,0,0,0.15)';
    const textElements = card.querySelectorAll('h1, p, span, a');
    textElements.forEach(el => el.style.color = '#222222');
    const img = card.querySelector('img');
    if (img) img.style.boxShadow = '0 6px 20px rgba(0,0,0,0.25)';
  }
}

document.addEventListener('DOMContentLoaded', updateProfileCardTheme);
document.addEventListener('click', function(e) {
  if (e.target.closest('#theme-toggle') || e.target.closest('.theme-toggle')) {
    setTimeout(updateProfileCardTheme, 50);
  }
});
window.addEventListener('storage', function(e) {
  if (e.key === 'pref-theme') updateProfileCardTheme();
});
</script>

{{< /rawhtml >}}
