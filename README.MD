```
<?php
$db = new PDO('sqlite:database.db');
$db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

$db->exec("
    CREATE TABLE IF NOT EXISTS ieraksti (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        nosaukums TEXT NOT NULL,
        apraksts TEXT,
        izveidots DATETIME DEFAULT CURRENT_TIMESTAMP
    )
");

if (isset($_GET['delete'])) {
    $stmt = $db->prepare("DELETE FROM ieraksti WHERE id = ?");
    $stmt->execute([(int)$_GET['delete']]);
    header("Location: index.php");
    exit;
}

if (isset($_POST['add'])) {
    $stmt = $db->prepare(
        "INSERT INTO ieraksti (nosaukums, apraksts) VALUES (?, ?)"
    );
    $stmt->execute([
        $_POST['nosaukums'],
        $_POST['apraksts']
    ]);
    header("Location: index.php");
    exit;
}

if (isset($_POST['edit'])) {
    $stmt = $db->prepare(
        "UPDATE ieraksti
         SET nosaukums = ?, apraksts = ?
         WHERE id = ?"
    );
    $stmt->execute([
        $_POST['nosaukums'],
        $_POST['apraksts'],
        $_POST['id']
    ]);
    header("Location: index.php");
    exit;
}

$edit = null;

if (isset($_GET['edit'])) {
    $stmt = $db->prepare("SELECT * FROM ieraksti WHERE id = ?");
    $stmt->execute([(int)$_GET['edit']]);
    $edit = $stmt->fetch(PDO::FETCH_ASSOC);
}

$ieraksti = $db->query(
    "SELECT * FROM ieraksti ORDER BY id DESC"
)->fetchAll(PDO::FETCH_ASSOC);
?>

<!DOCTYPE html>
<html lang="lv">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Ierakstu pārvaldības sistēma</title>

    <style>
        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            font-family: Arial, sans-serif;
            background: linear-gradient(135deg, #667eea, #764ba2);
            min-height: 100vh;
            padding: 40px 20px;
        }

        .container {
            max-width: 1000px;
            margin: auto;
        }

        h1 {
            color: white;
            text-align: center;
            margin-bottom: 10px;
        }

        .subtitle {
            text-align: center;
            color: #eee;
            margin-bottom: 30px;
        }

        .box {
            background: white;
            padding: 25px;
            border-radius: 15px;
            margin-bottom: 25px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.2);
        }

        h2 {
            margin-top: 0;
            color: #333;
        }

        label {
            display: block;
            margin-top: 15px;
            margin-bottom: 7px;
            font-weight: bold;
        }

        input,
        textarea {
            width: 100%;
            padding: 12px;
            border: 1px solid #ccc;
            border-radius: 8px;
            font-size: 16px;
        }

        textarea {
            min-height: 100px;
            resize: vertical;
        }

        button,
        .button {
            display: inline-block;
            border: none;
            border-radius: 8px;
            padding: 11px 18px;
            margin-top: 15px;
            color: white;
            text-decoration: none;
            cursor: pointer;
            font-size: 15px;
        }

        button {
            background: #667eea;
        }

        button:hover,
        .button:hover {
            opacity: 0.85;
        }

        .edit {
            background: #f39c12;
        }

        .delete {
            background: #e74c3c;
        }

        .cancel {
            background: #777;
        }

        table {
            width: 100%;
            border-collapse: collapse;
        }

        th {
            background: #667eea;
            color: white;
            padding: 13px;
            text-align: left;
        }

        td {
            padding: 13px;
            border-bottom: 1px solid #ddd;
        }

        tr:hover {
            background: #f5f5f5;
        }

        .empty {
            text-align: center;
            color: #777;
            padding: 30px;
        }

        @media (max-width: 700px) {
            body {
                padding: 20px 10px;
            }

            table,
            thead,
            tbody,
            th,
            td,
            tr {
                display: block;
            }

            thead {
                display: none;
            }

            tr {
                background: #fff;
                border: 1px solid #ddd;
                margin-bottom: 15px;
                border-radius: 8px;
                padding: 10px;
            }

            td {
                border: none;
                padding: 8px;
            }
        }
    </style>
</head>

<body>

<div class="container">

    <h1>📋 Ierakstu pārvaldības sistēma</h1>

    <div class="subtitle">
        PHP CRUD sistēma ierakstu pārvaldīšanai
    </div>

    <div class="box">

        <?php if ($edit): ?>

            <h2>✏️ Labot ierakstu</h2>

            <form method="POST">

                <input
                    type="hidden"
                    name="id"
                    value="<?= htmlspecialchars($edit['id']) ?>"
                >

                <label>Nosaukums</label>

                <input
                    type="text"
                    name="nosaukums"
                    required
                    value="<?= htmlspecialchars($edit['nosaukums']) ?>"
                >

                <label>Apraksts</label>

                <textarea name="apraksts"><?= htmlspecialchars($edit['apraksts']) ?></textarea>

                <button type="submit" name="edit">
                    Saglabāt
                </button>

                <a href="index.php" class="button cancel">
                    Atcelt
                </a>

            </form>

        <?php else: ?>

            <h2>➕ Pievienot ierakstu</h2>

            <form method="POST">

                <label>Nosaukums</label>

                <input
                    type="text"
                    name="nosaukums"
                    placeholder="Ievadi nosaukumu..."
                    required
                >

                <label>Apraksts</label>

                <textarea
                    name="apraksts"
                    placeholder="Ievadi aprakstu..."
                ></textarea>

                <button type="submit" name="add">
                    Pievienot ierakstu
                </button>

            </form>

        <?php endif; ?>

    </div>

    <div class="box">

        <h2>📁 Visi ieraksti</h2>

        <?php if (count($ieraksti) > 0): ?>

            <table>

                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Nosaukums</th>
                        <th>Apraksts</th>
                        <th>Datums</th>
                        <th>Darbības</th>
                    </tr>
                </thead>

                <tbody>

                <?php foreach ($ieraksti as $ieraksts): ?>

                    <tr>

                        <td>
                            <?= htmlspecialchars($ieraksts['id']) ?>
                        </td>

                        <td>
                            <strong>
                                <?= htmlspecialchars($ieraksts['nosaukums']) ?>
                            </strong>
                        </td>

                        <td>
                            <?= nl2br(htmlspecialchars($ieraksts['apraksts'])) ?>
                        </td>

                        <td>
                            <?= htmlspecialchars($ieraksts['izveidots']) ?>
                        </td>

                        <td>

                            <a
                                href="?edit=<?= $ieraksts['id'] ?>"
                                class="button edit"
                            >
                                Labot
                            </a>

                            <a
                                href="?delete=<?= $ieraksts['id'] ?>"
                                class="button delete"
                                onclick="return confirm('Vai tiešām vēlies dzēst šo ierakstu?')"
                            >
                                Dzēst
                            </a>

                        </td>

                    </tr>

                <?php endforeach; ?>

                </tbody>

            </table>

        <?php else: ?>

            <div class="empty">
                📭 Vēl nav neviena ieraksta.
            </div>

        <?php endif; ?>

    </div>

</div>

</body>
</html>
```