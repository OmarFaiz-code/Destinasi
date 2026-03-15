<?php
session_start();


if (!isset($_SESSION['logged_in'])) {
    header('Location: login.php');
    exit;
}

if (!isset($_SESSION['destinasi'])) {
    $_SESSION['destinasi'] = [
        [
            'id'        => 1,
            'nama'      => 'Gunung Rinjani',
            'lokasi'    => 'Lombok, NTB',
            'ketinggian'=> 3726,
            'kesulitan' => 'Berat',
            'deskripsi' => 'Gunung berapi aktif kedua tertinggi di Indonesia dengan danau kawah Segara Anak yang memukau.',
            'ditambahkan_oleh' => 'Admin Hiker',
            'gambar_emoji' => '🏔️',
            'warna'     => '#2d6a4f',
        ],
        [
            'id'        => 2,
            'nama'      => 'Gunung Semeru',
            'lokasi'    => 'Lumajang, Jawa Timur',
            'ketinggian'=> 3676,
            'kesulitan' => 'Berat',
            'deskripsi' => 'Atap Pulau Jawa dengan padang savana Oro-Oro Ombo dan puncak Mahameru yang legendaris.',
            'ditambahkan_oleh' => 'Admin Hiker',
            'gambar_emoji' => '⛰️',
            'warna'     => '#1b4332',
        ],
        [
            'id'        => 3,
            'nama'      => 'Gunung Prau',
            'lokasi'    => 'Dieng, Jawa Tengah',
            'ketinggian'=> 2590,
            'kesulitan' => 'Mudah',
            'deskripsi' => 'Populer untuk pemula karena jalur yang bersahabat dan panorama golden sunrise terbaik.',
            'ditambahkan_oleh' => 'Admin Hiker',
            'gambar_emoji' => '🌄',
            'warna'     => '#40916c',
        ],
        [
            'id'        => 4,
            'nama'      => 'Gunung Papandayan',
            'lokasi'    => 'Garut, Jawa Barat',
            'ketinggian'=> 2665,
            'kesulitan' => 'Sedang',
            'deskripsi' => 'Kawah aktif yang mengagumkan dengan hutan mati dan padang edelweiss Tegal Alun.',
            'ditambahkan_oleh' => 'Admin Hiker',
            'gambar_emoji' => '🌋',
            'warna'     => '#52b788',
        ],
    ];
}

$success = '';
$errors  = [];

// CREATE 
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['action']) && $_POST['action'] === 'tambah') {
    $nama       = trim($_POST['nama'] ?? '');
    $lokasi     = trim($_POST['lokasi'] ?? '');
    $ketinggian = trim($_POST['ketinggian'] ?? '');
    $kesulitan  = trim($_POST['kesulitan'] ?? '');
    $deskripsi  = trim($_POST['deskripsi'] ?? '');

    if (empty($nama))             $errors[] = 'Nama gunung wajib diisi.';
    if (empty($lokasi))           $errors[] = 'Lokasi wajib diisi.';
    if (!is_numeric($ketinggian) || $ketinggian <= 0) $errors[] = 'Ketinggian harus berupa angka positif.';
    if (empty($kesulitan))        $errors[] = 'Tingkat kesulitan wajib dipilih.';
    if (empty($deskripsi))        $errors[] = 'Deskripsi wajib diisi.';

    if (empty($errors)) {
        $warnaMap = ['Mudah' => '#52b788', 'Sedang' => '#40916c', 'Berat' => '#1b4332', 'Ekstrem' => '#0d2a1c'];
        $emoji    = ['Mudah' => '🌄', 'Sedang' => '⛰️', 'Berat' => '🏔️', 'Ekstrem' => '🌋'];

        $newId = count($_SESSION['destinasi']) > 0
                 ? max(array_column($_SESSION['destinasi'], 'id')) + 1 : 1;

        $_SESSION['destinasi'][] = [
            'id'        => $newId,
            'nama'      => $nama,
            'lokasi'    => $lokasi,
            'ketinggian'=> (int)$ketinggian,
            'kesulitan' => $kesulitan,
            'deskripsi' => $deskripsi,
            'ditambahkan_oleh' => $_SESSION['user_nama'],
            'gambar_emoji' => $emoji[$kesulitan] ?? '⛰️',
            'warna'     => $warnaMap[$kesulitan] ?? '#2f8f4a',
        ];
        $success = "Destinasi \"$nama\" berhasil ditambahkan!";
    }
}

// LOGOUT
if (isset($_GET['logout'])) {
    session_destroy();
    header('Location: login.php');
    exit;
}

$role  = $_SESSION['user_role'];
$nama  = $_SESSION['user_nama'];
$total = count($_SESSION['destinasi']);
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Destinasi — Hiker Best</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,700;0,900;1,700&family=Lato:wght@300;400;700&display=swap" rel="stylesheet">
    <style>
        *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

        :root {
            --green-dark: #1a4731;
            --green-mid:  #2f8f4a;
            --green-light:#4ab868;
            --cream:      #f5f0e8;
            --cream2:     #ece7da;
            --gold:       #c9a84c;
            --text:       #1a1a1a;
            --text-light: #666;
        }

        body {
            font-family: 'Lato', sans-serif;
            background: #f0ebe0;
            min-height: 100vh;
            color: var(--text);
        }

        /* ── NAVBAR ── */
        .navbar {
            background: var(--green-dark);
            padding: 0 32px;
            display: flex; align-items: center; justify-content: space-between;
            height: 64px;
            position: sticky; top: 0; z-index: 100;
            box-shadow: 0 4px 20px rgba(0,0,0,0.3);
        }
        .nav-brand {
            font-family: 'Playfair Display', serif;
            color: var(--cream); font-size: 1.4rem; font-weight: 900;
            letter-spacing: 2px; display: flex; align-items: center; gap: 10px;
        }
        .nav-brand span { color: var(--gold); }
        .nav-right { display: flex; align-items: center; gap: 20px; }
        .nav-user {
            display: flex; align-items: center; gap: 10px;
        }
        .avatar {
            width: 36px; height: 36px; border-radius: 50%;
            background: linear-gradient(135deg, var(--green-light), var(--gold));
            display: flex; align-items: center; justify-content: center;
            font-weight: 700; font-size: .85rem; color: white;
        }
        .user-info { line-height: 1.3; }
        .user-info .uname { font-size: .84rem; font-weight: 700; color: var(--cream); }
        .user-info .urole {
            font-size: .68rem; letter-spacing: 1.5px; text-transform: uppercase;
            color: var(--gold);
        }
        .btn-logout {
            padding: 7px 18px;
            background: rgba(255,255,255,0.08);
            border: 1px solid rgba(255,255,255,0.18);
            border-radius: 8px; color: rgba(245,240,232,0.8);
            font-family: 'Lato', sans-serif;
            font-size: .75rem; font-weight: 700; letter-spacing: 1.5px;
            text-transform: uppercase; text-decoration: none;
            transition: all .3s; cursor: pointer;
        }
        .btn-logout:hover { background: rgba(255,255,255,0.15); color: white; }

        /* ── HERO SECTION ── */
        .hero {
            background: linear-gradient(135deg, var(--green-dark) 0%, #0d2a1c 100%);
            padding: 56px 32px 48px;
            position: relative; overflow: hidden;
        }
        .hero::before {
            content: '';
            position: absolute; top: -80px; right: -80px;
            width: 300px; height: 300px; border-radius: 50%;
            background: rgba(74,184,104,0.08);
            border: 1px solid rgba(74,184,104,0.15);
        }
        .hero::after {
            content: '';
            position: absolute; bottom: -60px; left: 200px;
            width: 200px; height: 200px; border-radius: 50%;
            background: rgba(201,168,76,0.06);
        }
        .hero-inner { max-width: 1100px; margin: 0 auto; position: relative; z-index: 1; }
        .hero-eyebrow {
            font-size: .7rem; letter-spacing: 3px; text-transform: uppercase;
            color: var(--gold); margin-bottom: 10px;
        }
        .hero h2 {
            font-family: 'Playfair Display', serif;
            font-size: clamp(1.8rem, 4vw, 2.8rem); font-weight: 900;
            color: var(--cream); line-height: 1.15;
            margin-bottom: 14px;
        }
        .hero p { color: rgba(245,240,232,0.6); font-size: .9rem; max-width: 480px; line-height: 1.7; }
        .hero-stats {
            display: flex; gap: 32px; margin-top: 28px;
        }
        .stat { text-align: left; }
        .stat .num {
            font-family: 'Playfair Display', serif;
            font-size: 2rem; font-weight: 900; color: var(--gold);
        }
        .stat .lbl { font-size: .7rem; letter-spacing: 2px; text-transform: uppercase; color: rgba(245,240,232,0.45); }

        /* ── MAIN CONTENT ── */
        .main { max-width: 1100px; margin: 0 auto; padding: 40px 24px 80px; }

        /* ── ALERT BOXES ── */
        .alert-success {
            background: #f0fff4; border: 1px solid #a8ddb9; border-left: 4px solid var(--green-mid);
            border-radius: 10px; padding: 12px 16px;
            color: #1a6b35; font-size: .84rem; margin-bottom: 28px;
        }
        .alert-error {
            background: #fff0f0; border: 1px solid #ffcccc; border-left: 4px solid #e74c3c;
            border-radius: 10px; padding: 12px 16px;
            color: #c0392b; font-size: .84rem; margin-bottom: 28px;
        }
        .alert-error ul { margin-left: 16px; }
        .alert-error li { margin-bottom: 3px; }

        /* ── SECTION HEADER ── */
        .section-header {
            display: flex; align-items: flex-end; justify-content: space-between;
            margin-bottom: 24px;
        }
        .section-title {
            font-family: 'Playfair Display', serif;
            font-size: 1.6rem; font-weight: 700;
            color: var(--green-dark);
        }
        .section-count {
            font-size: .78rem; letter-spacing: 1.5px; text-transform: uppercase;
            color: var(--text-light);
        }

        /* ── GRID CREATE + READ ── */
        .layout-grid {
            display: grid;
            grid-template-columns: 340px 1fr;
            gap: 28px;
            align-items: start;
        }
        @media (max-width: 800px) {
            .layout-grid { grid-template-columns: 1fr; }
        }

        /* ── CREATE FORM CARD ── */
        .create-card {
            background: white;
            border-radius: 20px;
            padding: 28px 24px;
            box-shadow: 0 4px 24px rgba(0,0,0,0.06);
            border: 1px solid rgba(0,0,0,0.05);
            position: sticky; top: 80px;
        }
        .create-card h3 {
            font-family: 'Playfair Display', serif;
            font-size: 1.2rem; color: var(--green-dark);
            margin-bottom: 4px;
        }
        .create-card .sub { font-size: .78rem; color: var(--text-light); margin-bottom: 20px; }

        .field { margin-bottom: 16px; }
        .field label {
            display: block; font-size: .68rem; font-weight: 700;
            letter-spacing: 1.5px; text-transform: uppercase;
            color: var(--green-dark); margin-bottom: 6px;
        }
        .field input, .field select, .field textarea {
            width: 100%; padding: 10px 13px;
            border: 1.5px solid #e0e0e0; border-radius: 9px;
            font-family: 'Lato', sans-serif; font-size: .88rem;
            color: var(--text); outline: none; transition: .3s;
            background: #fafafa;
        }
        .field textarea { resize: vertical; min-height: 80px; }
        .field input:focus, .field select:focus, .field textarea:focus {
            border-color: var(--green-mid);
            box-shadow: 0 0 0 3px rgba(47,143,74,.1);
            background: white;
        }

        .badge-role {
            display: inline-block; padding: 3px 10px;
            border-radius: 20px; font-size: .68rem; font-weight: 700;
            letter-spacing: 1px; text-transform: uppercase;
            margin-bottom: 16px;
        }
        .badge-admin { background: rgba(201,168,76,.15); color: #8a6000; border: 1px solid rgba(201,168,76,.3); }
        .badge-user  { background: rgba(47,143,74,.12); color: var(--green-dark); border: 1px solid rgba(47,143,74,.2); }

        .btn-create {
            width: 100%; padding: 12px;
            background: linear-gradient(135deg, var(--green-mid), var(--green-dark));
            color: white; border: none; border-radius: 10px;
            font-family: 'Lato', sans-serif;
            font-size: .8rem; font-weight: 700; letter-spacing: 2px;
            text-transform: uppercase; cursor: pointer; transition: all .3s;
            margin-top: 4px;
        }
        .btn-create:hover { transform: translateY(-2px); box-shadow: 0 8px 20px rgba(47,143,74,.4); }

        .locked-msg {
            text-align: center; padding: 20px 0;
            color: var(--text-light); font-size: .84rem;
        }
        .locked-msg .lock-icon { font-size: 2rem; margin-bottom: 8px; }

        /* ── DESTINATION CARDS ── */
        .dest-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
            gap: 20px;
        }

        .dest-card {
            background: white; border-radius: 18px;
            overflow: hidden;
            box-shadow: 0 4px 20px rgba(0,0,0,0.07);
            border: 1px solid rgba(0,0,0,0.04);
            transition: transform .3s, box-shadow .3s;
            animation: fadeIn .5s ease both;
        }
        .dest-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 12px 36px rgba(0,0,0,0.14);
        }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(16px); } to { opacity: 1; transform: none; } }

        .dest-header {
            height: 110px;
            display: flex; align-items: center; justify-content: center;
            font-size: 3.5rem;
            position: relative; overflow: hidden;
        }
        .dest-header::after {
            content: '';
            position: absolute; inset: 0;
            background: linear-gradient(to bottom, transparent 50%, rgba(0,0,0,0.18));
        }

        .kesulitan-badge {
            position: absolute; top: 10px; right: 10px;
            font-size: .62rem; font-weight: 700; letter-spacing: 1px;
            text-transform: uppercase; padding: 3px 9px;
            border-radius: 20px; z-index: 1;
        }
        .k-mudah   { background: rgba(82,183,136,.9); color: white; }
        .k-sedang  { background: rgba(64,145,108,.9); color: white; }
        .k-berat   { background: rgba(27,67,50,.9); color: var(--gold); }
        .k-ekstrem { background: rgba(13,42,28,.9); color: #ff8c42; }

        .dest-body { padding: 18px; }
        .dest-nama {
            font-family: 'Playfair Display', serif;
            font-size: 1.1rem; font-weight: 700;
            color: var(--green-dark); margin-bottom: 4px;
        }
        .dest-lokasi {
            font-size: .75rem; color: var(--text-light);
            margin-bottom: 10px; display: flex; align-items: center; gap: 4px;
        }
        .dest-ketinggian {
            font-size: .78rem; font-weight: 700; color: var(--green-mid);
            margin-bottom: 10px;
        }
        .dest-desc {
            font-size: .8rem; color: #555; line-height: 1.65;
            margin-bottom: 14px;
            display: -webkit-box; -webkit-line-clamp: 3;
            -webkit-box-orient: vertical; overflow: hidden;
        }
        .dest-footer {
            border-top: 1px solid #f0f0f0;
            padding-top: 10px;
            font-size: .7rem; color: #aaa;
        }

        /* ── EMPTY STATE ── */
        .empty-state {
            text-align: center; padding: 60px 20px;
            color: var(--text-light);
            grid-column: 1 / -1;
        }
        .empty-state .icon { font-size: 4rem; margin-bottom: 14px; }
        .empty-state p { font-size: .9rem; }
    </style>
</head>
<body>

<!-- NAVBAR -->
<nav class="navbar">
    <div class="nav-brand">🏔️ HIKER <span>BEST</span></div>
    <div class="nav-right">
        <div class="nav-user">
            <div class="avatar"><?= strtoupper(substr($nama, 0, 1)) ?></div>
            <div class="user-info">
                <div class="uname"><?= htmlspecialchars($nama) ?></div>
                <div class="urole"><?= strtoupper($role) ?></div>
            </div>
        </div>
        <a href="?logout=1" class="btn-logout">Keluar</a>
    </div>
</nav>

<!-- HERO -->
<div class="hero">
    <div class="hero-inner">
        <div class="hero-eyebrow">🌿 Eksplorasi Nusantara</div>
        <h2>Destinasi<br><em>Pendakian Terbaik</em></h2>
        <p>Temukan gunung-gunung indah di seluruh Indonesia. Rencanakan pendakian impianmu bersama Hiker Best.</p>
        <div class="hero-stats">
            <div class="stat">
                <div class="num"><?= $total ?></div>
                <div class="lbl">Destinasi</div>
            </div>
            <div class="stat">
                <div class="num">17+</div>
                <div class="lbl">Provinsi</div>
            </div>
            <div class="stat">
                <div class="num">3K+</div>
                <div class="lbl">Pendaki</div>
            </div>
        </div>
    </div>
</div>

<!-- MAIN -->
<div class="main">

    <?php if ($success): ?>
        <div class="alert-success">✅ <?= htmlspecialchars($success) ?></div>
    <?php endif; ?>
    <?php if (!empty($errors)): ?>
        <div class="alert-error">
            <strong>⚠️ Terdapat kesalahan:</strong>
            <ul><?php foreach ($errors as $e) echo "<li>" . htmlspecialchars($e) . "</li>"; ?></ul>
        </div>
    <?php endif; ?>

    <div class="section-header">
        <div class="section-title">Kelola Destinasi</div>
        <div class="section-count"><?= $total ?> destinasi terdaftar</div>
    </div>

    <div class="layout-grid">

        <!-- ── CREATE (kiri) ── -->
        <div class="create-card">
            <h3>Tambah Destinasi</h3>
            <p class="sub">Daftarkan gunung baru ke sistem</p>

            <?php if ($role === 'admin'): ?>
                <span class="badge-role badge-admin">👑 Admin — Akses Penuh</span>
                <form method="POST" action="destinasi.php">
                    <input type="hidden" name="action" value="tambah">
                    <div class="field">
                        <label>Nama Gunung</label>
                        <input type="text" name="nama" placeholder="cth. Gunung Merbabu" required>
                    </div>
                    <div class="field">
                        <label>Lokasi</label>
                        <input type="text" name="lokasi" placeholder="cth. Magelang, Jawa Tengah" required>
                    </div>
                    <div class="field">
                        <label>Ketinggian (mdpl)</label>
                        <input type="number" name="ketinggian" placeholder="cth. 3142" min="100" required>
                    </div>
                    <div class="field">
                        <label>Tingkat Kesulitan</label>
                        <select name="kesulitan" required>
                            <option value="" disabled selected>— Pilih —</option>
                            <option value="Mudah">Mudah</option>
                            <option value="Sedang">Sedang</option>
                            <option value="Berat">Berat</option>
                            <option value="Ekstrem">Ekstrem</option>
                        </select>
                    </div>
                    <div class="field">
                        <label>Deskripsi Singkat</label>
                        <textarea name="deskripsi" placeholder="Ceritakan keindahan gunung ini..." required></textarea>
                    </div>
                    <button type="submit" class="btn-create">+ Tambah Destinasi</button>
                </form>

            <?php else: ?>
                <span class="badge-role badge-user">🥾 User — Mode Baca</span>
                <div class="locked-msg">
                    <div class="lock-icon">🔒</div>
                    <p>Hanya <strong>Admin</strong> yang dapat menambahkan destinasi baru.</p>
                    <p style="margin-top:6px;font-size:.75rem;color:#bbb;">Silakan login sebagai admin untuk mengakses fitur ini.</p>
                </div>
            <?php endif; ?>
        </div>

        <!-- ── READ (kanan) ── -->
        <div>
            <div class="dest-grid">
                <?php if (empty($_SESSION['destinasi'])): ?>
                    <div class="empty-state">
                        <div class="icon">🗻</div>
                        <p>Belum ada destinasi. Tambahkan yang pertama!</p>
                    </div>
                <?php else: ?>
                    <?php foreach (array_reverse($_SESSION['destinasi']) as $i => $d): ?>
                    <?php
                        $kClass = 'k-' . strtolower($d['kesulitan']);
                    ?>
                    <div class="dest-card" style="animation-delay: <?= $i * 0.07 ?>s">
                        <div class="dest-header" style="background: <?= htmlspecialchars($d['warna']) ?>;">
                            <span class="kesulitan-badge <?= $kClass ?>"><?= htmlspecialchars($d['kesulitan']) ?></span>
                            <?= $d['gambar_emoji'] ?>
                        </div>
                        <div class="dest-body">
                            <div class="dest-nama"><?= htmlspecialchars($d['nama']) ?></div>
                            <div class="dest-lokasi">📍 <?= htmlspecialchars($d['lokasi']) ?></div>
                            <div class="dest-ketinggian">▲ <?= number_format($d['ketinggian']) ?> mdpl</div>
                            <div class="dest-desc"><?= htmlspecialchars($d['deskripsi']) ?></div>
                            <div class="dest-footer">
                                Ditambahkan oleh: <?= htmlspecialchars($d['ditambahkan_oleh']) ?>
                            </div>
                        </div>
                    </div>
                    <?php endforeach; ?>
                <?php endif; ?>
            </div>
        </div>

    </div>
</div>

</body>
</html>
