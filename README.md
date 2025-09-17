# 📊 Dashboard Modular PHP + Bootstrap 5

Este projeto é um painel administrativo modular desenvolvido em PHP, com layout responsivo baseado em Bootstrap 5. Ele permite integrar múltiplos projetos de forma organizada, escalável e segura, com carregamento dinâmico de páginas, estilos, scripts e funções específicas por módulo. Esse documento serve como guia técnico e introdutório para desenvolvedores que desejam entender, instalar e expandir o sistema.

---

## 🚀 Funcionalidades

- ✅ Layout com sidebar retrátil
- ✅ Carregamento dinâmico de módulos e páginas via URL
- ✅ Inclusão automática de arquivos CSS, JS e funções PHP por módulo
- ✅ Menu lateral gerado dinamicamente com base nos módulos existentes
- ✅ Persistência do estado do sidebar com `localStorage`
- ✅ Integração com banco de dados via PDO
- ✅ Estrutura escalável para novos projetos e funcionalidades

---

## 🗂️ Estrutura de Diretórios

```plaintext
dashboard/
├── index.php              # Arquivo principal
├── header.php             # Cabeçalho HTML + estilos globais
├── sidebar.php            # Menu lateral dinâmico
├── footer.php             # Scripts globais + persistência do sidebar
├── modules/               # Diretório de módulos
│   ├── home/
│   │   ├── index.php
│   │   ├── includes/
│   │   │   └── functions.php
│   │   └── assets/
│   │       ├── css/
│   │       │   └── style.css
│   │       └── js/
│   │           └── script.js
│   ├── relatorios/
│   │   ├── index.php
│   │   ├── includes/
│   │   │   └── db.php
│   │   └── assets/
│   │       ├── css/
│   │       │   └── chart.css
│   │       └── js/
│   │           └── chart.js
│   └── config/
│       ├── index.php
│       ├── includes/
│       │   └── settings.php
│       └── assets/
│           ├── css/
│           │   └── config.css
│           └── js/
│               └── config.js
```

---

## ⚙️ index.php (principal)

```php
<?php
session_start();

function sanitize($input) {
  $input = (new SplFileInfo($input))->getFilename();
  return preg_match('/^[a-zA-Z0-9_-]+$/', $input) ? $input : null;
}

$modulo = sanitize($_GET['modulo'] ?? 'home');
$pagina = sanitize($_GET['pagina'] ?? 'index');

$modulosDisponiveis = array_filter(scandir('modules'), fn($item) =>
  is_dir("modules/$item") && !in_array($item, ['.', '..', 'erro404'])
);

if (!in_array($modulo, $modulosDisponiveis)) {
  $modulo = 'erro404';
  $pagina = 'index';
}

$conteudo = "modules/{$modulo}/{$pagina}.php";

include 'header.php';
?>

<div class="d-flex">
  <?php include 'sidebar.php'; ?>

  <main class="flex-grow-1 p-4" id="main">
    <?php
      $includesDir = "modules/{$modulo}/includes/";
      if (is_dir($includesDir)) {
        foreach (scandir($includesDir) as $file) {
          if (pathinfo($file, PATHINFO_EXTENSION) === 'php') {
            include_once "{$includesDir}{$file}";
          }
        }
      }

      if (file_exists($conteudo)) {
        include $conteudo;
      } else {
        echo "<h2>Erro 404</h2><p>Página não encontrada.</p>";
      }
    ?>
  </main>
</div>

<?php
$cssDir = "modules/{$modulo}/assets/css/";
if (is_dir($cssDir)) {
  foreach (scandir($cssDir) as $css) {
    if (pathinfo($css, PATHINFO_EXTENSION) === 'css') {
      echo "<link rel='stylesheet' href='{$cssDir}{$css}'>";
    }
  }
}

$jsDir = "modules/{$modulo}/assets/js/";
if (is_dir($jsDir)) {
  foreach (scandir($jsDir) as $js) {
    if (pathinfo($js, PATHINFO_EXTENSION) === 'js') {
      echo "<script src='{$jsDir}{$js}'></script>";
    }
  }
}

include 'footer.php';
?>
```

---

## 📋 sidebar.php (menu lateral dinâmico)

```php
<nav id="sidebar" class="bg-dark text-white p-3">
  <div class="d-flex justify-content-between align-items-center mb-3">
    <span class="fs-5 fw-bold">Painel</span>
    <button class="toggle-btn" onclick="toggleSidebar()">☰</button>
  </div>
  <ul class="nav flex-column">
    <?php
    foreach ($modulosDisponiveis as $mod) {
      if (file_exists("modules/{$mod}/index.php")) {
        echo "<li class='nav-item'>
                <a href='index.php?modulo={$mod}&pagina=index' class='nav-link text-white'>
                  📁 <span>" . ucfirst($mod) . "</span>
                </a>
              </li>";
      }
    }
    ?>
  </ul>
</nav>
```

---

## 🧩 footer.php (scripts globais + persistência do sidebar)

```html
<script>
  document.addEventListener("DOMContentLoaded", function () {
    const sidebar = document.getElementById('sidebar');
    const main = document.getElementById('main');
    const estado = localStorage.getItem('sidebarEstado');

    if (estado === 'collapsed') {
      sidebar.classList.add('collapsed');
      main.classList.add('collapsed');
    }
  });

  function toggleSidebar() {
    const sidebar = document.getElementById('sidebar');
    const main = document.getElementById('main');
    sidebar.classList.toggle('collapsed');
    main.classList.toggle('collapsed');

    const estado = sidebar.classList.contains('collapsed') ? 'collapsed' : 'expanded';
    localStorage.setItem('sidebarEstado', estado);
  }
</script>
</body>
</html>
```

---

## 🧪 Exemplo de módulo: `modules/relatorios/index.php`

```php
<?php include_once 'includes/db.php'; ?>

<h2>Relatórios</h2>
<table class="table table-bordered">
  <thead>
    <tr><th>ID</th><th>Nome</th><th>Valor</th></tr>
  </thead>
  <tbody>
    <?php
    $stmt = $pdo->query("SELECT id, nome, valor FROM relatorios");
    while ($row = $stmt->fetch()) {
      echo "<tr><td>{$row['id']}</td><td>{$row['nome']}</td><td>{$row['valor']}</td></tr>";
    }
    ?>
  </tbody>
</table>
```

---

## 🗄️ Exemplo de conexão: `modules/relatorios/includes/db.php`

```php
<?php
$host = 'localhost';
$db   = 'dashboard';
$user = 'root';
$pass = '';
$charset = 'utf8mb4';

try {
  $pdo = new PDO("mysql:host=$host;dbname=$db;charset=$charset", $user, $pass);
  $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
  die("Erro na conexão: " . $e->getMessage());
}
```

---

## 📦 Requisitos

- Servidor com PHP 7.4+
- Banco de dados MySQL (opcional)
- Bootstrap 5 via CDN
- Permissões de leitura nas pastas e arquivos

---
