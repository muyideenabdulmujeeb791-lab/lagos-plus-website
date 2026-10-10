<?php
// Lagos Plus in one PHP file. Requires PHP with PDO_SQLite enabled.
session_start();
$authMessage = '';
$activeMode = 'signup';
$playerName = $_SESSION['user_name'] ?? 'Player';
$db = null;
try {
    if (!class_exists('PDO') || !in_array('sqlite', PDO::getAvailableDrivers(), true)) {
        throw new RuntimeException('This hosting account does not have SQLite support enabled. Please use PHP hosting with PDO_SQLite enabled.');
    }
    $db = new PDO('sqlite:' . __DIR__ . '/lagos-plus.sqlite');
    $db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    $db->exec('CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY AUTOINCREMENT, full_name TEXT NOT NULL, email TEXT NOT NULL UNIQUE, password_hash TEXT NOT NULL, created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP)');
} catch (Throwable $e) {
    $authMessage = $e->getMessage();
}
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['auth_action'])) {
    $activeMode = (string)$_POST['auth_action'];
    if ($activeMode === 'logout') {
        $_SESSION = [];
        if (ini_get('session.use_cookies')) {
            $params = session_get_cookie_params();
            setcookie(session_name(), '', time() - 42000, $params['path'], $params['domain'], $params['secure'], $params['httponly']);
        }
        session_destroy();
        header('Location: index.php');
        exit;
    }
    $email = strtolower(trim((string)($_POST['email'] ?? '')));
    $password = (string)($_POST['password'] ?? '');
    $fullName = trim((string)($_POST['full_name'] ?? ''));
    if (!$db) {
        // The database setup error is shown on the login screen.
    } elseif ($activeMode === 'forgot') {
        $authMessage = 'Password reset email is not configured yet. Your account has not been changed. Email-based recovery needs a mail service to be configured on the server.';
    } elseif (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $authMessage = 'Please enter a valid email address.';
    } elseif (strlen($password) < 8) {
        $authMessage = 'Please use a password with at least 8 characters.';
    } elseif ($activeMode === 'signup') {
        if ($fullName === '' || strlen($fullName) > 180) {
            $authMessage = 'Please enter your name (up to 60 characters).';
        } else {
            try {
                $stmt = $db->prepare('INSERT INTO users (full_name, email, password_hash) VALUES (:name, :email, :password)');
                $stmt->execute([':name' => $fullName, ':email' => $email, ':password' => password_hash($password, PASSWORD_DEFAULT)]);
                session_regenerate_id(true);
                $_SESSION['user_id'] = (int)$db->lastInsertId();
                $_SESSION['user_name'] = $fullName;
                header('Location: index.php');
                exit;
            } catch (PDOException $e) {
                $authMessage = (stripos($e->getMessage(), 'unique') !== false)
                    ? 'That email already has an account. Choose Log in instead.'
                    : 'Could not create your account. Please try again.';
            }
        }
    } elseif ($activeMode === 'login') {
        $stmt = $db->prepare('SELECT id, full_name, password_hash FROM users WHERE email = :email LIMIT 1');
        $stmt->execute([':email' => $email]);
        $user = $stmt->fetch(PDO::FETCH_ASSOC);
        if ($user && password_verify($password, $user['password_hash'])) {
            session_regenerate_id(true);
            $_SESSION['user_id'] = (int)$user['id'];
            $_SESSION['user_name'] = $user['full_name'];
            header('Location: index.php');
            exit;
        }
        $authMessage = 'Email or password is incorrect. Check your details or create an account.';
    } else {
        $activeMode = 'signup';
        $authMessage = 'Please choose Sign up or Log in.';
    }
}
$isLoggedIn = isset($_SESSION['user_id']);
$playerName = $_SESSION['user_name'] ?? 'Player';
if ($authMessage === '' && !$isLoggedIn && !$db) {
    $authMessage = 'The account database could not start. Check that PHP SQLite support is enabled.';
}
?>
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Lagos Plus — Your Life, Your Lagos</title>
<meta name="theme-color" content="#0c1220">
<meta name="description" content="Lagos Plus is a Nigerian-inspired life simulation game. Work, explore, manage your needs, and build your virtual life.">
<style>
:root{--bg:#0c1220;--panel:#151f32;--panel2:#1b2940;--text:#f4f7fb;--muted:#aab7ca;--green:#65e0a3;--gold:#ffd166;--blue:#78b7ff;--red:#ff7d86;--line:#2a3a54}
*{box-sizing:border-box}body{margin:0;background:radial-gradient(ellipse at 20% 0%,#1c3150 0,transparent 42%),var(--bg);color:var(--text);font:16px/1.45 system-ui,-apple-system,Segoe UI,Roboto,Arial,sans-serif}
button,input{font:inherit}button{cursor:pointer;border:0;border-radius:12px;padding:12px 14px;font-weight:750;transition:transform .12s,filter .12s}button:hover{filter:brightness(1.08);transform:translateY(-1px)}button:focus-visible,input:focus-visible{outline:3px solid var(--gold);outline-offset:2px}
.app{max-width:1180px;margin:auto;padding:22px}.topbar{display:flex;justify-content:space-between;align-items:center;gap:16px;margin-bottom:20px}.brand{display:flex;align-items:center;gap:12px}.logo{display:grid;place-items:center;width:48px;height:48px;border-radius:15px;background:linear-gradient(135deg,#ffd166,#65e0a3);color:#112033;font-size:25px;font-weight:950}.brand h1{font-size:23px;letter-spacing:-.7px;margin:0}.brand p{margin:2px 0 0;color:var(--muted);font-size:13px}.top-actions{display:flex;gap:8px;flex-wrap:wrap}.secondary{background:#25344c;color:var(--text);border:1px solid #344761}.primary{background:var(--green);color:#092218}.gold{background:var(--gold);color:#30230a}.danger{background:#512b3a;color:#ffdce0}
.hero{display:grid;grid-template-columns:1.15fr .85fr;gap:18px;margin-bottom:18px}.hero-copy,.city-card,.panel,.stat{background:linear-gradient(145deg,rgba(27,41,64,.97),rgba(18,28,45,.97));border:1px solid var(--line);border-radius:22px}.hero-copy{padding:27px;position:relative;overflow:hidden}.eyebrow{color:var(--green);font-size:12px;font-weight:850;letter-spacing:1.5px;text-transform:uppercase}.hero h2{font-size:clamp(29px,4vw,44px);line-height:1.06;letter-spacing:-1.7px;margin:12px 0}.hero h2 span{color:var(--gold)}.hero-copy p{color:var(--muted);max-width:530px}.hero-buttons{display:flex;gap:10px;flex-wrap:wrap;margin-top:20px}.city-card{min-height:250px;padding:20px;display:flex;flex-direction:column;justify-content:space-between;overflow:hidden;background:linear-gradient(180deg,rgba(21,39,59,.2),rgba(13,23,38,.7)),linear-gradient(135deg,#24556b,#263d5b 48%,#d18b4b)}.skyline{display:flex;align-items:end;gap:8px;height:130px;border-bottom:3px solid #f4c16b;position:relative}.building{flex:1;min-width:17px;background:linear-gradient(90deg,#182b45,#355b70);border-radius:4px 4px 0 0;position:relative}.building:after{content:"";position:absolute;inset:12px 5px 5px;background:repeating-linear-gradient(0deg,transparent 0 12px,#f5d17b 13px 17px),repeating-linear-gradient(90deg,transparent 0 10px,#f5d17b 11px 14px);opacity:.65}.building:nth-child(1){height:48%}.building:nth-child(2){height:75%}.building:nth-child(3){height:58%}.building:nth-child(4){height:95%;background:#1c314a}.building:nth-child(5){height:67%}.building:nth-child(6){height:84%}.building:nth-child(7){height:42%}.danfo{font-size:36px;position:absolute;bottom:-18px;left:28%;filter:drop-shadow(0 4px 0 rgba(0,0,0,.2))}.city-caption{display:flex;justify-content:space-between;gap:12px;align-items:end}.city-caption strong{font-size:20px}.city-caption small{color:#e6edf5}
.stats{display:grid;grid-template-columns:repeat(4,1fr);gap:12px;margin-bottom:18px}.stat{padding:15px}.stat-head{display:flex;justify-content:space-between;gap:8px;color:var(--muted);font-size:13px}.stat-value{font-size:25px;font-weight:850;margin:5px 0 9px}.bar{height:8px;border-radius:99px;background:#0c1524;overflow:hidden}.bar span{display:block;height:100%;width:50%;border-radius:99px;transition:width .25s}.money{color:var(--green)}.energy{background:var(--gold)}.happiness{background:#ff8dc7}.health{background:#78b7ff}
.layout{display:grid;grid-template-columns:1.25fr .75fr;gap:18px}.panel{padding:20px}.panel-title{display:flex;justify-content:space-between;align-items:center;gap:12px;margin-bottom:14px}.panel-title h3{margin:0;font-size:19px}.tag{font-size:11px;font-weight:800;color:var(--green);background:#1d493d;padding:5px 9px;border-radius:99px}.location-grid{display:grid;grid-template-columns:repeat(2,1fr);gap:11px}.location{padding:15px;border:1px solid var(--line);border-radius:16px;background:#111b2b;display:flex;gap:12px;align-items:flex-start}.loc-icon{width:43px;height:43px;display:grid;place-items:center;border-radius:13px;background:#24354d;font-size:24px;flex-shrink:0}.location h4{margin:0 0 3px;font-size:15px}.location p{margin:0;color:var(--muted);font-size:12px}.location button{margin-top:10px;background:#293b54;color:#fff;padding:8px 10px;font-size:12px}.side-stack{display:grid;gap:18px;align-content:start}.job-row,.task-row{display:flex;align-items:center;justify-content:space-between;gap:12px;padding:12px 0;border-bottom:1px solid var(--line)}.job-row:last-child,.task-row:last-child{border-bottom:0}.job-row strong,.task-row strong{display:block;font-size:14px}.job-row small,.task-row small{display:block;color:var(--muted);font-size:12px;margin-top:3px}.job-row button,.task-row button{white-space:nowrap;background:#244735;color:#baffd7;padding:9px 11px;font-size:12px}.task-row button{background:#293b54;color:#fff}.log{max-height:240px;overflow:auto;display:grid;gap:9px}.log-entry{padding:10px 12px;border-radius:12px;background:#111b2b;color:#c8d4e4;font-size:13px}.log-entry:first-child{border:1px solid #315341;color:#e6fff0}.feature-grid{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:12px}.feature-card{padding:15px;border:1px solid var(--line);border-radius:15px;background:#111b2b}.feature-card h4{margin:0 0 5px}.feature-card p{margin:0 0 10px;color:var(--muted);font-size:13px}.feature-card button{background:#244735;color:#baffd7;padding:9px 11px;font-size:12px}.feature-card button:disabled{opacity:.45;cursor:not-allowed}.mission-item{padding:12px 0;border-bottom:1px solid var(--line)}.mission-item:last-child{border-bottom:0}.mission-item p{margin:4px 0 8px;color:var(--muted);font-size:13px}.mission-item button{background:#293b54;color:#fff;padding:8px 10px;font-size:12px}.inventory-line{color:var(--muted);font-size:13px;margin:6px 0}.footer{color:#8292a8;text-align:center;font-size:12px;padding:24px 0 6px}.notice{padding:12px 14px;border-radius:12px;background:#2b2940;color:#f2e7ff;margin-bottom:15px;font-size:13px}
.modal-backdrop{position:fixed;inset:0;background:#030711cc;display:grid;place-items:center;padding:20px;z-index:5}.modal{width:min(440px,100%);background:#17243a;border:1px solid #3b4f6e;border-radius:22px;padding:25px;box-shadow:0 25px 80px #0008}.modal h2{margin:0 0 8px}.modal p{color:var(--muted);font-size:14px}.modal label{display:block;margin:16px 0 6px;font-size:13px;font-weight:750}.modal input{width:100%;padding:12px;border-radius:11px;border:1px solid #3b4f6e;background:#0c1627;color:white}.modal-actions{display:flex;gap:10px;margin-top:18px}.modal-actions button{flex:1}.hidden{display:none!important}
@media(max-width:800px){.hero,.layout{grid-template-columns:1fr}.city-card{min-height:220px}.stats{grid-template-columns:repeat(2,1fr)}}@media(max-width:480px){.feature-grid{grid-template-columns:1fr}.app{padding:13px}.topbar{align-items:flex-start}.brand h1{font-size:20px}.top-actions{justify-content:end}.top-actions button{padding:9px;font-size:12px}.hero-copy{padding:20px}.stats{gap:8px}.stat{padding:12px}.stat-value{font-size:22px}.location-grid{grid-template-columns:1fr}.panel{padding:15px}.hero h2{font-size:34px}}

/* Account UI preview and animated airport expansion */
.auth-actions{display:flex;gap:8px;flex-wrap:wrap}.auth-panel{margin-bottom:18px}.auth-tabs{display:flex;gap:8px;flex-wrap:wrap}.auth-form{display:grid;gap:10px;margin-top:12px}.auth-form input,.auth-form select{width:100%;padding:12px;border-radius:11px;border:1px solid #3b4f6e;background:#0c1627;color:white}.auth-note{font-size:12px;color:var(--gold);margin:10px 0 0}.airport-scene{position:relative;overflow:hidden;min-height:150px;border-radius:16px;padding:16px;background:linear-gradient(#193c5b 0 55%,#3b4b59 55% 65%,#182637 65%);border:1px solid #36516a}.airport-scene:before{content:"✈️";position:absolute;right:12%;top:20px;font-size:38px;animation:planeFloat 5s ease-in-out infinite}.airport-runway{position:absolute;left:0;right:0;bottom:22px;border-top:3px dashed #e7dca9;opacity:.75}.crowd{position:absolute;left:0;right:0;bottom:27px;height:50px;display:flex;align-items:end;justify-content:space-around;white-space:nowrap}.traveller{font-size:26px;display:inline-block;animation:walk 4s linear infinite}.traveller:nth-child(2n){animation-delay:-1.4s;animation-duration:5s}.traveller:nth-child(3n){animation-delay:-2.5s;animation-duration:4.4s}.airport-caption{position:relative;z-index:1;font-weight:800;text-shadow:0 2px 5px #000}.airport-controls{display:flex;gap:10px;align-items:center;flex-wrap:wrap;margin-top:12px}.airport-controls select{background:#111b2b;color:#fff;border:1px solid var(--line);padding:11px;border-radius:10px;flex:1;min-width:180px}@keyframes walk{0%{transform:translateX(-10px)}50%{transform:translateX(12px)}100%{transform:translateX(-10px)}}@keyframes planeFloat{0%,100%{transform:translate(0,0) rotate(-3deg)}50%{transform:translate(-28px,-8px) rotate(3deg)}}


.auth-gate{min-height:100vh;display:grid;place-items:center;padding:22px;background:radial-gradient(ellipse at 20% 0%,#1c3150 0,transparent 42%),var(--bg)}
.auth-card{width:min(480px,100%);padding:28px;background:linear-gradient(145deg,#1b2940,#121c2d);border:1px solid var(--line);border-radius:24px;box-shadow:0 25px 80px #0006}
.auth-logo{margin-bottom:18px}.auth-card h1{font-size:clamp(28px,5vw,38px);margin:6px 0}.auth-intro{color:var(--muted);margin:0 0 20px}.auth-tabs{display:grid;grid-template-columns:1fr 1fr 1.35fr;gap:7px}.auth-tabs button{padding:10px 7px;font-size:13px}.auth-form{display:grid;gap:9px;margin-top:18px}.auth-form label{font-size:13px;font-weight:750}.auth-form input{width:100%;padding:12px;border-radius:11px;border:1px solid #3b4f6e;background:#0c1627;color:white}.auth-form button{margin-top:8px}.auth-message{padding:11px;border-radius:10px;background:#111b2b;color:#c8d4e4;font-size:13px}.auth-small{color:var(--gold);font-size:11px}

</style>
</head>
<body>
<?php if (!$isLoggedIn): ?>
<div id="authGate" class="auth-gate">
  <section class="auth-card">
    <div class="logo auth-logo">L+</div>
    <p class="eyebrow">YOUR LIFE. YOUR CHOICES. YOUR LAGOS.</p>
    <h1>Welcome to Lagos Plus</h1>
    <p class="auth-intro">Create an account or log in before entering your game dashboard.</p>
    <div class="auth-tabs">
      <button type="button" class="primary" data-gate-tab="signup">Sign up</button>
      <button type="button" class="secondary" data-gate-tab="login">Log in</button>
      <button type="button" class="secondary" data-gate-tab="forgot">Forgot password?</button>
    </div>
    <form id="gateForm" class="auth-form" method="post" action="index.php">
      <input type="hidden" id="authAction" name="auth_action" value="<?= htmlspecialchars($activeMode, ENT_QUOTES, 'UTF-8') ?>">
      <label id="gateNameLabel" for="gateName">Full name</label>
      <input id="gateName" name="full_name" autocomplete="name" placeholder="Your full name" maxlength="60">
      <label for="gateEmail">Email address</label>
      <input id="gateEmail" name="email" type="email" autocomplete="email" placeholder="you@example.com" required>
      <label id="gatePasswordLabel" for="gatePassword">Password</label>
      <input id="gatePassword" name="password" type="password" autocomplete="new-password" minlength="8" placeholder="At least 8 characters">
      <button id="gateSubmit" class="primary" type="submit">Create account</button>
    </form>
    <p id="gateMessage" class="auth-message" role="status"><?= htmlspecialchars($authMessage !== '' ? $authMessage : 'Create an account or log in to enter Lagos Plus. Your password is stored as a secure hash.', ENT_QUOTES, 'UTF-8') ?></p>
    <p class="auth-small">Use a unique password. Account data is stored by the PHP server.</p>
  </section>
</div>
<?php else: ?>
<div class="app" id="gameApp">
  <header class="topbar">
    <div class="brand"><div class="logo">L+</div><div><h1>Lagos Plus</h1><p>Your life. Your choices. Your Lagos.</p></div></div>
    <div class="top-actions"><button class="secondary" id="saveBtn">Save game</button><button class="secondary" id="resetBtn">New life ↻</button><button class="danger" id="logoutBtn" type="button">Log out ↪</button></div>
  </header>

  <section class="hero">
    <div class="hero-copy">
      <div class="eyebrow">Welcome to your next chapter</div>
      <h2>Make your move.<br><span>Build your Lagos.</span></h2>
      <p>Start with big dreams and a little cash. Find work, explore the city, look after yourself, and turn your choices into your own story.</p>
      <div class="hero-buttons"><button class="primary" id="workNow">💼 Go to work</button><button class="gold" id="exploreNow">🚌 Explore Lagos</button></div>
    </div>
    <div class="city-card" aria-label="Illustration of a city skyline and a yellow bus">
      <div class="skyline"><div class="building"></div><div class="building"></div><div class="building"></div><div class="building"></div><div class="building"></div><div class="building"></div><div class="building"></div><div class="danfo">🚌</div></div>
      <div class="city-caption"><div><strong id="greeting">Eko welcomes you!</strong><br><small id="locationLine">You're in Yaba, Lagos</small></div><div style="text-align:right"><small>DAY</small><div style="font-size:25px;font-weight:900" id="dayNum">01</div></div></div>
    </div>
  </section>

  <div class="notice" id="welcomeNotice">✨ Your story starts here. Take a job, grab a meal, or explore somewhere new. Your progress is saved on this device.</div>

  <section class="stats" aria-label="Character statistics">
    <div class="stat"><div class="stat-head"><span>💵 Wallet</span><span>NAIRA</span></div><div class="stat-value money" id="cash">₦5,000</div><div class="bar"><span id="cashBar" style="background:var(--green);width:45%"></span></div></div>
    <div class="stat"><div class="stat-head"><span>⚡ Energy</span><span id="energyLabel">Ready</span></div><div class="stat-value" id="energyValue">80 / 100</div><div class="bar"><span class="energy" id="energyBar" style="width:80%"></span></div></div>
    <div class="stat"><div class="stat-head"><span>💖 Happiness</span><span id="happyLabel">Doing well</span></div><div class="stat-value" id="happyValue">70 / 100</div><div class="bar"><span class="happiness" id="happyBar" style="width:70%"></span></div></div>
    <div class="stat"><div class="stat-head"><span>🩺 Health</span><span id="healthLabel">Healthy</span></div><div class="stat-value" id="healthValue">90 / 100</div><div class="bar"><span class="health" id="healthBar" style="width:90%"></span></div></div>
  </section>

  <main class="layout">
    <section class="panel">
      <div class="panel-title"><h3>Explore the city</h3><span class="tag">YOUR MAP</span></div>
      <div class="location-grid">
        <article class="location"><div class="loc-icon">🏙️</div><div><h4>Yaba</h4><p>Tech, learning and new opportunities.</p><button data-place="Yaba">Visit · ₦100</button></div></article>
        <article class="location"><div class="loc-icon">🌊</div><div><h4>Lekki</h4><p>Coastal vibes and a taste of luxury.</p><button data-place="Lekki">Visit · ₦500</button></div></article>
        <article class="location"><div class="loc-icon">🛍️</div><div><h4>Balogun Market</h4><p>Busy stalls, bargains and business ideas.</p><button data-place="Balogun Market">Visit · ₦200</button></div></article>
        <article class="location"><div class="loc-icon">🌳</div><div><h4>Freedom Park</h4><p>Slow down, relax and reset.</p><button data-place="Freedom Park">Visit · ₦300</button></div></article>
      </div>
    </section>

    <div class="side-stack">
      <section class="panel">
        <div class="panel-title"><h3>Earn a living</h3><span class="tag">JOBS</span></div>
        <div class="job-row"><div><strong>🛵 Dispatch rider</strong><small>Earn ₦2,500 · Uses 25 energy</small></div><button data-job="dispatch">Work</button></div>
        <div class="job-row"><div><strong>💻 Junior tech assistant</strong><small>Earn ₦3,500 · Uses 30 energy</small></div><button data-job="tech">Work</button></div>
        <div class="job-row"><div><strong>🛒 Shop assistant</strong><small>Earn ₦1,800 · Uses 18 energy</small></div><button data-job="shop">Work</button></div>
      </section>
      <section class="panel">
        <div class="panel-title"><h3>Look after yourself</h3><span class="tag">DAILY LIFE</span></div>
        <div class="task-row"><div><strong>🍲 Buy a meal</strong><small>Cost ₦700 · Restores health and energy</small></div><button id="eatBtn">Eat</button></div>
        <div class="task-row"><div><strong>🛏️ Get some sleep</strong><small>Advance to tomorrow · Restore energy</small></div><button id="sleepBtn">Sleep</button></div>
        <div class="task-row"><div><strong>🎧 Enjoy some free time</strong><small>Cost ₦300 · Boost happiness</small></div><button id="funBtn">Relax</button></div>
      </section>
      <section class="panel">
        <div class="panel-title"><h3>Your story</h3><span class="tag">LIVE LOG</span></div>
        <div class="log" id="log"><div class="log-entry">👋 Welcome to Lagos Plus! Your new life begins in Yaba.</div></div>
      </section>
    </div>
    <section class="panel" id="airportPanel">
      <div class="panel-title"><h3>✈️ Murtala Muhammed Airport</h3><span class="tag">TRAVEL & CROWD</span></div>
      <div class="airport-scene"><div class="airport-caption">Lagos International Airport · Departures</div><div class="crowd" aria-label="Animated travellers walking"><span class="traveller">🧍🏾</span><span class="traveller">🧳</span><span class="traveller">🚶🏽</span><span class="traveller">🧍🏻</span><span class="traveller">🧳</span><span class="traveller">🚶🏿</span><span class="traveller">🧍🏽</span></div><div class="airport-runway"></div></div>
      <p style="color:var(--muted);font-size:13px">Travellers move around the terminal. Choose a destination and take a flight in this game prototype.</p>
      <div class="airport-controls"><select id="flightDestination" aria-label="Flight destination"><option value="Abuja">Abuja · ₦18,000</option><option value="Accra">Accra · ₦45,000</option><option value="London">London · ₦350,000</option><option value="Port Harcourt">Port Harcourt · ₦28,000</option></select><button class="gold" id="flyBtn">✈️ Book flight</button></div>
    </section>
    <section class="panel">
      <div class="panel-title"><h3>🏠 Your home & garage</h3><span class="tag">PROPERTY</span></div>
      <div class="feature-grid">
        <article class="feature-card"><h4>🏚️ Shared room in Yaba</h4><p>Affordable starter home. Price: ₦8,000</p><button id="buyRoomBtn">Rent starter room</button></article>
        <article class="feature-card"><h4>🏡 Lekki apartment</h4><p>A nicer place to live. Price: ₦45,000</p><button id="buyApartmentBtn">Get apartment</button></article>
        <article class="feature-card"><h4>🚲 Used bicycle</h4><p>Own a ride and travel cheaper. Price: ₦12,000</p><button id="buyBikeBtn">Buy bicycle</button></article>
        <article class="feature-card"><h4>🚗 Used car</h4><p>Travel in comfort. Price: ₦250,000</p><button id="buyCarBtn">Buy car</button></article>
      </div>
      <p class="inventory-line" id="propertyStatus">Home: none · Ride: none</p>
    </section>
    <section class="panel">
      <div class="panel-title"><h3>🛒 Lagos shops</h3><span class="tag">MARKETPLACE</span></div>
      <div class="feature-grid">
        <article class="feature-card"><h4>🍲 Jollof rice</h4><p>₦1,200 · Energy +25, health +5</p><button data-buy="meal">Buy meal</button></article>
        <article class="feature-card"><h4>👟 Sneakers</h4><p>₦8,000 · Happiness +12</p><button data-buy="sneakers">Buy sneakers</button></article>
        <article class="feature-card"><h4>📱 Smartphone</h4><p>₦35,000 · Happiness +18</p><button data-buy="phone">Buy phone</button></article>
        <article class="feature-card"><h4>🎮 Game console</h4><p>₦90,000 · Happiness +25</p><button data-buy="console">Buy console</button></article>
      </div>
      <p class="inventory-line" id="inventoryStatus">Your items: none yet</p>
    </section>
    <section class="panel">
      <div class="panel-title"><h3>🎯 Missions</h3><span class="tag">EARN REWARDS</span></div>
      <div id="missionList"></div>
    </section>
  </main>
  <footer class="footer">Lagos Plus · A fan-made life simulation prototype · Not affiliated with any existing Lagos Life brand.</footer>
</div>
<form id="logoutForm" method="post" action="index.php" hidden><input type="hidden" name="auth_action" value="logout"></form>
<?php endif; ?>

<script>
(() => {
  const SAVE_KEY = "lagosPlusSave_v1";
  const AUTH_PLAYER_NAME = <?= json_encode($playerName, JSON_HEX_TAG | JSON_HEX_APOS | JSON_HEX_AMP | JSON_HEX_QUOT) ?>;
  const initialState = () => ({name:AUTH_PLAYER_NAME,cash:5000,energy:80,happiness:70,health:90,day:1,location:"Yaba",started:true,home:"None",ride:"None",inventory:[],workCount:0,visitCount:0,missionsClaimed:[]});
  let game = initialState();
  const $ = id => document.getElementById(id);
  const clamp = n => Math.max(0, Math.min(100, Math.round(n)));
  const naira = n => "₦" + Math.floor(n).toLocaleString("en-NG");

  // The server (PHP) handles signup/login. These controls only switch form modes.
  let gateMode = <?= json_encode($activeMode) ?>;
  const setGateMode = mode => {
    gateMode = mode;
    const signup = mode === "signup", forgot = mode === "forgot";
    $("authAction").value = mode;
    $("gateNameLabel").classList.toggle("hidden", !signup);
    $("gateName").classList.toggle("hidden", !signup);
    $("gateName").required = signup;
    $("gatePasswordLabel").classList.toggle("hidden", forgot);
    $("gatePassword").classList.toggle("hidden", forgot);
    $("gatePassword").required = !forgot;
    $("gatePassword").autocomplete = signup ? "new-password" : "current-password";
    $("gateSubmit").textContent = signup ? "Create account & enter" : forgot ? "Request password reset" : "Log in & enter";
    document.querySelectorAll("[data-gate-tab]").forEach(button => button.classList.toggle("primary", button.dataset.gateTab === mode));
    document.querySelectorAll("[data-gate-tab]").forEach(button => button.classList.toggle("secondary", button.dataset.gateTab !== mode));
  };
  document.querySelectorAll("[data-gate-tab]").forEach(button => button.addEventListener("click", () => setGateMode(button.dataset.gateTab)));
  if (document.getElementById("authGate")) setGateMode(gateMode);

  function log(message) {
    const entry = document.createElement("div");
    entry.className = "log-entry";
    entry.textContent = message;
    $("log").prepend(entry);
    while ($("log").children.length > 12) $("log").lastElementChild.remove();
  }
  function update() {
    $("cash").textContent = naira(game.cash);
    $("energyValue").textContent = `${game.energy} / 100`;
    $("happyValue").textContent = `${game.happiness} / 100`;
    $("healthValue").textContent = `${game.health} / 100`;
    $("energyBar").style.width = game.energy + "%";
    $("happyBar").style.width = game.happiness + "%";
    $("healthBar").style.width = game.health + "%";
    $("cashBar").style.width = Math.min(100, game.cash / 100) + "%";
    $("dayNum").textContent = String(game.day).padStart(2,"0");
    $("greeting").textContent = game.name === "Player" ? "Eko welcomes you!" : `Welcome, ${game.name}!`;
    $("locationLine").textContent = `You're in ${game.location}, Lagos`;
    $("energyLabel").textContent = game.energy > 60 ? "Ready" : game.energy > 25 ? "Getting tired" : "Exhausted";
    $("happyLabel").textContent = game.happiness > 65 ? "Doing well" : game.happiness > 30 ? "Could be better" : "Feeling low";
    $("healthLabel").textContent = game.health > 65 ? "Healthy" : game.health > 30 ? "Needs care" : "Unwell";
    $("workNow").disabled = game.energy < 18;
    const ps = $("propertyStatus");
    if (ps) ps.textContent = `Home: ${game.home || "None"} · Ride: ${game.ride || "None"}`;
    const inv = $("inventoryStatus");
    if (inv) inv.textContent = "Your items: " + ((game.inventory || []).length ? game.inventory.join(", ") : "none yet");
    renderMissions();
  }
  function save(showMessage = true) {
    try {
      localStorage.setItem(SAVE_KEY, JSON.stringify(game));
      if (showMessage) log("💾 Game saved on this device.");
    } catch (e) {
      log("⚠️ Saving is unavailable in this browser session.");
    }
  }
  function load() {
    try {
      const raw = localStorage.getItem(SAVE_KEY);
      if (!raw) return false;
      const parsed = JSON.parse(raw);
      if (!parsed || typeof parsed !== "object" || typeof parsed.cash !== "number") return false;
      game = {...initialState(), ...parsed};
      return true;
    } catch (e) { return false; }
  }
  function spend(amount) {
    if (game.cash < amount) { log(`❌ You need ${naira(amount)} for that. Find work to earn more.`); return false; }
    game.cash -= amount;
    return true;
  }
  function work(kind) {
    const jobs = {
      dispatch:{name:"dispatch riding",pay:2500,energy:25,happy:-3},
      tech:{name:"a junior tech assistant shift",pay:3500,energy:30,happy:2},
      shop:{name:"a shop assistant shift",pay:1800,energy:18,happy:0}
    };
    const job = jobs[kind];
    if (!job) return;
    if (game.energy < job.energy) { log("😴 You're too tired to work this shift. Get some sleep first."); return; }
    game.cash += job.pay;
    game.workCount = (game.workCount || 0) + 1;
    game.energy = clamp(game.energy - job.energy);
    game.happiness = clamp(game.happiness + job.happy);
    game.health = clamp(game.health - 2);
    log(`💼 You completed ${job.name} and earned ${naira(job.pay)}.`);
    update(); save(false);
  }
  function explore() {
    if (!spend(150)) return;
    const places = ["Yaba","Lekki","Balogun Market","Freedom Park"];
    const place = places[Math.floor(Math.random() * places.length)];
    game.location = place;
    game.energy = clamp(game.energy - 8);
    game.happiness = clamp(game.happiness + 8);
    log(`🚌 You explored ${place}. A change of scene lifted your mood.`);
    update(); save(false);
  }
  function visit(place) {
    const costs = {"Yaba":100,"Lekki":500,"Balogun Market":200,"Freedom Park":300};
    const cost = costs[place] ?? 200;
    if (!spend(cost)) return;
    game.location = place;
    game.visitCount = (game.visitCount || 0) + 1;
    game.energy = clamp(game.energy - 5);
    game.happiness = clamp(game.happiness + (place === "Freedom Park" ? 12 : 6));
    if (place === "Balogun Market" && Math.random() < .25) {
      game.cash += 500;
      log("🛍️ You spotted a small resale opportunity at Balogun Market and made ₦500.");
    } else {
      log(`📍 You visited ${place}. The trip cost ${naira(cost)}.`);
    }
    update(); save(false);
  }
  function eat() {
    if (!spend(700)) return;
    game.energy = clamp(game.energy + 22);
    game.health = clamp(game.health + 10);
    game.happiness = clamp(game.happiness + 3);
    log("🍲 You enjoyed a good meal. Your energy and health improved.");
    update(); save(false);
  }
  function sleep() {
    game.day += 1;
    game.energy = clamp(game.energy + 55);
    game.health = clamp(game.health + 8);
    game.happiness = clamp(game.happiness + 2);
    log(`🌙 You got some rest. Day ${game.day} begins with renewed energy.`);
    update(); save(false);
  }
  function relax() {
    if (!spend(300)) return;
    game.happiness = clamp(game.happiness + 16);
    game.energy = clamp(game.energy + 5);
    log("🎧 You took time for yourself. Sometimes a little break is worth it.");
    update(); save(false);
  }
  function buyProperty(kind) {
    const options = {
      room: {cost:8000, type:"home", value:"Shared room in Yaba", message:"You rented a shared room in Yaba."},
      apartment: {cost:45000, type:"home", value:"Lekki apartment", message:"Welcome to your new Lekki apartment!"},
      bike: {cost:12000, type:"ride", value:"Used bicycle", message:"You bought a bicycle. Short trips are easier now."},
      car: {cost:250000, type:"ride", value:"Used car", message:"Congratulations on your used car!"}
    };
    const item = options[kind]; if (!item) return;
    if (game[item.type] && game[item.type] !== "None") { log(`ℹ️ You already have a ${item.type}.`); return; }
    if (!spend(item.cost)) return;
    game[item.type] = item.value; log("🎉 " + item.message); update(); save(false);
  }
  function buyShopItem(kind) {
    const items = {
      meal:{name:"Jollof rice",cost:1200,energy:25,health:5,happiness:0},
      sneakers:{name:"Sneakers",cost:8000,happiness:12},
      phone:{name:"Smartphone",cost:35000,happiness:18},
      console:{name:"Game console",cost:90000,happiness:25}
    };
    const item = items[kind]; if (!item) return;
    if (!spend(item.cost)) return;
    if (item.energy) game.energy = clamp(game.energy + item.energy);
    if (item.health) game.health = clamp(game.health + item.health);
    if (item.happiness) game.happiness = clamp(game.happiness + item.happiness);
    game.inventory = game.inventory || []; game.inventory.push(item.name);
    log(`🛍️ You bought ${item.name} for ${naira(item.cost)}.`); update(); save(false);
  }
  function renderMissions() {
    const target = $("missionList"); if (!target) return;
    const missions = [
      {id:"firstjob", title:"First paycheque", desc:"Complete 1 job shift", done:(game.workCount||0)>=1, reward:1000},
      {id:"explorer", title:"Know your city", desc:"Visit 3 places around Lagos", done:(game.visitCount||0)>=3, reward:1500},
      {id:"homeowner", title:"Make it your own", desc:"Get a home", done:!!game.home && game.home!=="None", reward:2000},
      {id:"wheels", title:"Get moving", desc:"Buy a bicycle or car", done:!!game.ride && game.ride!=="None", reward:2500}
    ];
    game.missionsClaimed = game.missionsClaimed || [];
    target.replaceChildren();
    missions.forEach(m => {
      const row = document.createElement("div"); row.className="mission-item";
      const title = document.createElement("strong"); title.textContent = `${m.done ? "✅" : "🎯"} ${m.title}`;
      const desc = document.createElement("p"); desc.textContent = `${m.desc} · Reward: ${naira(m.reward)}`;
      const btn = document.createElement("button");
      const claimed = game.missionsClaimed.includes(m.id);
      btn.textContent = claimed ? "Reward claimed" : m.done ? "Claim reward" : "Not completed yet";
      btn.disabled = !m.done || claimed;
      btn.addEventListener("click", () => { if (!m.done || game.missionsClaimed.includes(m.id)) return; game.missionsClaimed.push(m.id); game.cash += m.reward; log(`🏆 Mission complete: ${m.title}. You received ${naira(m.reward)}.`); update(); save(false); });
      row.append(title, desc, btn); target.append(row);
    });
  }
  const flightPrices = {"Abuja":18000,"Accra":45000,"London":350000,"Port Harcourt":28000};
  $("flyBtn").addEventListener("click", () => {
    const destination = $("flightDestination").value;
    const fare = flightPrices[destination];
    if (!spend(fare)) return;
    game.location = destination;
    game.day += 1;
    game.energy = clamp(game.energy - 12);
    game.happiness = clamp(game.happiness + 10);
    log(`✈️ You flew to ${destination}. Ticket cost ${naira(fare)}.`);
    update(); save(false);
  });
  let authMode = "signup";
  document.querySelectorAll("[data-auth-tab]").forEach(btn => btn.addEventListener("click", () => {
    authMode = btn.dataset.authTab;
    const signup = authMode === "signup", forgot = authMode === "forgot";
    $("authNameLabel").classList.toggle("hidden", !signup);
    $("authName").classList.toggle("hidden", !signup);
    $("authPasswordLabel").classList.toggle("hidden", forgot);
    $("authPassword").classList.toggle("hidden", forgot);
    $("authPassword").required = !forgot;
    $("authSubmit").textContent = signup ? "Create account" : forgot ? "Request reset link" : "Log in";
    $("authNote").textContent = "Preview only: real account security and password reset require a server and database. Do not enter a real password here.";
  }));
  $("authForm").addEventListener("submit", e => {
    e.preventDefault();
    $("authNote").textContent = authMode === "forgot"
      ? "Password reset is not connected yet. To send a real reset email, we must add a secure backend and email service."
      : "This form is a visual prototype only; it does not create or verify accounts. Next, we can connect a secure backend and database.";
  });
  $("buyRoomBtn").addEventListener("click", () => buyProperty("room"));
  $("buyApartmentBtn").addEventListener("click", () => buyProperty("apartment"));
  $("buyBikeBtn").addEventListener("click", () => buyProperty("bike"));
  $("buyCarBtn").addEventListener("click", () => buyProperty("car"));
  document.querySelectorAll("[data-buy]").forEach(button => button.addEventListener("click", () => buyShopItem(button.dataset.buy)));

  $("workNow").addEventListener("click", () => work("dispatch"));
  $("exploreNow").addEventListener("click", explore);
  $("eatBtn").addEventListener("click", eat);
  $("sleepBtn").addEventListener("click", sleep);
  $("funBtn").addEventListener("click", relax);
  document.querySelectorAll("[data-job]").forEach(button => button.addEventListener("click", () => work(button.dataset.job)));
  document.querySelectorAll("[data-place]").forEach(button => button.addEventListener("click", () => visit(button.dataset.place)));
  $("saveBtn").addEventListener("click", () => save(true));
  $("logoutBtn").addEventListener("click", () => {
    save(false);
    $("logoutForm").submit();
  });
  $("resetBtn").addEventListener("click", () => {
    if (!confirm("Start a new life? This replaces your saved Lagos Plus progress on this device.")) return;
    game = initialState();
    $("log").replaceChildren();
    log("✨ A fresh start! Welcome to your new life in Yaba.");
    update(); save(false);
  });
  const hadSave = load();
  update();
  if (hadSave && game.started) {
    $("welcomeNotice").textContent = `👋 Welcome back, ${game.name}. Your saved life is ready to continue.`;
    log(`↩️ Welcome back, ${game.name}. Your progress was loaded.`);
  }
})();
</script>
</body>
</html>
