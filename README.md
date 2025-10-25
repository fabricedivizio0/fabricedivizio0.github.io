<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>LIGHTRI - Trieur de cartes</title>
<style>
body { font-family: monospace; background: #f5f5f5; padding: 20px; }
textarea { width: 100%; height: 400px; }
button { margin: 10px 0; padding: 10px 20px; font-size: 16px; }
</style>
</head>
<body>
<h1>LIGHTRI - Trieur de cartes</h1>
<input type="file" id="fileInput" accept=".txt" multiple>
<br>
<button id="processBtn">Traiter le(s) fichier(s)</button>
<button id="downloadBtn">Télécharger le résultat</button>
<br>
<textarea id="output" placeholder="Résultat ici..." readonly></textarea>

<script>
async function processFiles(files) {
    let finalOutput = '';

    for (const file of files) {
        const text = await file.text();
        const lines = text.split(/\r?\n/);

        for (const line of lines) {
            if (!line.trim()) continue;
            const fields = line.split('|').map(f => f.trim());
            if (fields.length < 12) continue;

            const [CardNumber, ExpMonth, ExpYear, CVV, BankField, FullName, Address, City, City2, Zip, Phone, Email] = fields;

            const cleanCard = CardNumber.replace(/[\s-]/g, '');
            const BIN = cleanCard.slice(0,6);

            // Appel API Binlist
            let Scheme='', CardType='', Brand='', BankName='', CountryName='';
            try {
                const res = await fetch(`https://lookup.binlist.net/${BIN}`);
                if (res.ok) {
                    const data = await res.json();
                    Scheme = data.scheme || 'Non spécifié';
                    CardType = data.type || 'Non spécifié';
                    Brand = data.brand || 'Non spécifié';
                    BankName = (data.bank && data.bank.name) || 'Non spécifié';
                    CountryName = (data.country && data.country.name) || 'Non spécifié';
                }
            } catch(e) {
                console.warn('Binlist API error for BIN', BIN);
            }

            const block = `💳 + 9 NEW CARD
└ ${cleanCard}

🏦 Informations Personnelles
├ 🕵️ Nom complet : ${FullName}
├ 🏠 Adresse : ${Address}
├ 📮 Zip : ${Zip}
├ 🏢 Ville : ${City}
├ 📞 Numéro de téléphone : ${Phone}
└ 📧 Email : ${Email}

🏦 Carte de Paiement
├ 🍒 Titulaire : ${FullName}
├ 💳 Numéro de carte : ${cleanCard}
├ 💳 Bin : ${BIN}
├ 💳 Réseau : ${Scheme}
├ 💳 Banque : ${BankName} (${CountryName})
├ 💳 Niveau : ${Brand}
├ 💳 Type : ${CardType}
├ 📅 Expiration : ${ExpMonth}/${ExpYear}
└ 🔒 Cryptogramme visuel : ${CVV}

`;
            finalOutput += block + '\n';
        }
    }
    document.getElementById('output').value = finalOutput;
}

document.getElementById('processBtn').addEventListener('click', async () => {
    const files = document.getElementById('fileInput').files;
    if (!files.length) { alert('Choisissez au moins un fichier .txt'); return; }
    document.getElementById('output').value = 'Traitement en cours...';
    await processFiles(files);
    alert('Traitement terminé !');
});

document.getElementById('downloadBtn').addEventListener('click', () => {
    const text = document.getElementById('output').value;
    if (!text) { alert('Aucun résultat à télécharger'); return; }
    const blob = new Blob([text], {type:'text/plain'});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `LIGHTRI_Result_${Date.now()}.txt`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
});
</script>
</body>
</html>
