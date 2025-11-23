Instructions :

Retournez dans votre Google Sheet > Extensions > Apps Script.

Effacez TOUT le code précédent.

Collez ce nouveau code.

Faites Déployer > Nouveau déploiement (Assurez-vous de créer une nouvelle version, sinon les changements ne seront pas pris en compte).

JavaScript

function doGet(e) {
  return handleRequest(e);
}

function handleRequest(e) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  // IMPORTANT : Assurez-vous que le nom de l'onglet est bien "Feuille 1" ou changez-le ici
  const sheet = ss.getSheets()[0]; // Prend automatiquement la première feuille
  
  const action = e.parameter.action;
  const query = e.parameter.query;
  const title = e.parameter.title;
  const supportParam = e.parameter.support; // Ex: "DVD", "Blu-Ray"
  
  let result = { status: 'error', message: 'Action inconnue' };
  
  // Mapping des colonnes (A=0, B=1, C=2, D=3) basé sur votre fichier CSV
  const COLUMNS = {
    'LASERDISC': 0, // Colonne A
    'DVD': 1,       // Colonne B
    'Blu-Ray': 2,   // Colonne C
    'à acheter': 3  // Colonne D
  };

  const HEADERS = ['LASERDISC', 'DVD', 'Blu-Ray', 'à acheter'];
  
  if (action === "search") {
    // Récupérer toutes les données de la plage (on suppose max 2000 lignes pour être large)
    // On commence à la ligne 2 pour sauter les en-têtes
    const lastRow = sheet.getLastRow();
    if (lastRow > 1) {
      const data = sheet.getRange(2, 1, lastRow - 1, 4).getValues();
      const found = [];
      const lowerQuery = query.toLowerCase();

      // On parcourt chaque ligne
      for (let i = 0; i < data.length; i++) {
        // On parcourt chaque colonne de la ligne (0 à 3)
        for (let col = 0; col < 4; col++) {
          const cellValue = String(data[i][col]).trim();
          if (cellValue && cellValue.toLowerCase().includes(lowerQuery)) {
            found.push({
              title: cellValue,
              support: HEADERS[col] // Retourne le nom de la colonne (ex: "DVD")
            });
          }
        }
      }
      result = { status: 'success', data: found };
    } else {
      result = { status: 'success', data: [] };
    }
    
  } else if (action === "add") {
    const colIndex = COLUMNS[supportParam];
    
    if (colIndex !== undefined) {
      // Trouver la première cellule vide dans cette colonne spécifique
      // On récupère toute la colonne (+1 car getRange commence à 1)
      const columnData = sheet.getRange(1, colIndex + 1, sheet.getMaxRows(), 1).getValues();
      
      let targetRow = -1;
      // On cherche la première case vide
      for (let i = 0; i < columnData.length; i++) {
        if (!columnData[i][0]) {
          targetRow = i + 1;
          break;
        }
      }
      
      // Si on n'a pas trouvé de case vide (feuille pleine), on prend la dernière + 1
      if (targetRow === -1) targetRow = columnData.length + 1;
      
      sheet.getRange(targetRow, colIndex + 1).setValue(title);
      result = { status: 'success', message: 'Film ajouté dans ' + supportParam };
    } else {
      result = { status: 'error', message: 'Support inconnu' };
    }
  }
  
  return ContentService
    .createTextOutput(JSON.stringify(result))
    .setMimeType(ContentService.MimeType.JSON);
}
