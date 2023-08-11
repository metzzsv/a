aFileDialog - Guide de l'utilisateur
====================================

## 1. Utilisation basique

### Installation

Pour utiliser _aFileDialog_, dans votre logiciel, il faut suivre les suivants pas:

**1)** Ajouter une r�f�rence au projet de la librairie (situ� dans le dossier [library](../library/)). �tant donn� que _aFileDialog_ est un _projet de librairie d'Android_, il ne peut pas �tre compil� et distribu� comme un fichier binaire (comme, par exemple, un fichier JAR), � cause de �a on doit faire la r�f�rence au dossier du projet (avec le code source), en lieu de faire la r�f�rence � un fichier JAR. 

**2)** D�clarer la activit� _FileChooserActivity_ dans le fichier de manifeste. On peut faire �a en ajoutant les lignes suivantes entre les �tiquettes `<application></application>`:
	
```xml
    <activity android:name="ar.com.daidalos.afiledialog.FileChooserActivity" />
```

*3)* D�clarer le permis _READ_EXTERNAL_STORAGE_ dans le fichier de manifeste, si on a besoin d�acc�der � la carte SD (et si on utilise Android 4.1 ou sup�rieur).

```xml
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

**4)** Sur Android 6.0 et versions ult�rieures, en plus d'indiquer la permission sur le fichier _manifest_, vous devez �galement demander aux utilisateurs d'approuver l'autorisation lors de l'ex�cution. Par exemple:

```java
if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
    // If permission is not granted, ask it.
    ActivityCompat.requestPermissions(this,
            new String[]{Manifest.permission.READ_EXTERNAL_STORAGE},
            MY_PERMISSIONS_REQUEST_READ_EXTERNAL_STORAGE);
}
```

> Vous pouvez trouvar plus d'information dans https://developer.android.com/training/permissions/requesting.

Pour plus d'information sur comment faire une r�f�rence � un projet de librairie et comment d�clarer une activit� dans le fichier de manifeste, vous pouvez lire la [documentation officielle d'Android](https://developer.android.com/sdk/installing/create-project.html#ReferencingLibraryModule).

###  1.2. Afficher le s�lecteur de fichiers 

Si on veut utiliser un _Dialog_ pour afficher le s�lecteur de fichiers, on peut utiliser les lignes suivantes:

```java
    FileChooserDialog dialog = new FileChooserDialog(this);
    dialog.show();
```
	
Si on veut utiliser une _Activity_ pour afficher le s�lecteur de fichiers, alors on peut utiliser ces lignes de code:

```java
    Intent intent = new Intent(this, FileChooserActivity.class);
    this.startActivityForResult(intent, 0);
```
	
Par d�faut, le s�lecteur de fichiers montre le racine de la carte SD, n�anmoins, on peut changer le dossier que le s�lecteur montre la premi�re fois qu'il est ouvert.

Si on utilise un _Dialog_ pour afficher le s�lecteur, alors on doit appeler le m�thode _loadFolder()_ de la classe _FileChooserDialog_:

```java
    FileChooserDialog dialog = new FileChooserDialog(this);
    dialog.loadFolder(Environment.getExternalStorageDirectory() + "/Download/");
    dialog.show();
```
	
Si on utilise une _Activity_ pour afficher le s�lecteur, alors on doit ajouter un extra au _Intent_ avec le nom _FileChooserActivity.INPUT_START_FOLDER_ et le chemin du dossier comme le valeur:

```java
    Intent intent = new Intent(this, FileChooserActivity.class);
    intent.putExtra(FileChooserActivity.INPUT_START_FOLDER, Environment.getExternalStorageDirectory() + "/Download/");
    this.startActivityForResult(intent, 0);
``` 

###  1.3. Retrouver le fichier s�lectionn� 

� fin de retrouver le fichier, repr�sente comme un objet _File_, choisi par l'utilisateur, si on utilise un _Dialog_ pour afficher le s�lecteur, il faut appeler le m�thode  _addListener()_, de la classe _FileChooserDialog_, en utilisant comme param�tre une impl�mentation de la interface _FileChooserDialog.OnFileSelectedListener_:

```java
    dialog.addListener(new FileChooserDialog.OnFileSelectedListener() {
        public void onFileSelected(Dialog source, File file) {
            source.hide();
            Toast toast = Toast.makeText(source.getContext(), "File selected: " + file.getName(), Toast.LENGTH_LONG);
            toast.show();
		}
		public void onFileSelected(Dialog source, File folder, String name) {
            source.hide();
            Toast toast = Toast.makeText(source.getContext(), "File created: " + folder.getName() + "/" + name, Toast.LENGTH_LONG);
            toast.show();
		}
	});
```
	
L'interface _FileChooserDialog.OnFileSelectedListener_ d�fini deux m�thodes: _onFileSelected(Dialog source, File file)_, qui est appel� quand un fichier est s�lectionn�; et _onFileSelected(Dialog source, File folder, String name)_, qui est appel� quand un fichier est cr��.

Il faut tenir en compte, qu'est responsabilit� de l'application de fermer le s�lecteur de fichiers, une fois que un fichier a �t� s�lectionn�. De cette mani�re, si l'utilisateur s�lectionne un fichier incorrecte, l�application peut montrer un message d'erreur et ne pas fermer le s�lecteur de fichiers, � fin de permettre � l'utilisateur de s�lectionner autre fichier.

Si on utilise une _Activity_ pour afficher le s�lecteur de fichiers, alors la _Activity_ que appelle le s�lecteur doit impl�menter le m�thode _onActivityResult()_:

```java
	@Override
	protected void onActivityResult(int requestCode, int resultCode, Intent data) {
	    if (resultCode == Activity.RESULT_OK) {
	    	boolean fileCreated = false;
	    	String filePath = "";
	    	
	    	Bundle bundle = data.getExtras();
	        if(bundle != null)
	        {
	        	if(bundle.containsKey(FileChooserActivity.OUTPUT_NEW_FILE_NAME)) {
	        		fileCreated = true;
	        		File folder = (File) bundle.get(FileChooserActivity.OUTPUT_FILE_OBJECT);
	        		String name = bundle.getString(FileChooserActivity.OUTPUT_NEW_FILE_NAME);
	        		filePath = folder.getAbsolutePath() + "/" + name;
	        	} else {
	        		fileCreated = false;
	        		File file = (File) bundle.get(FileChooserActivity.OUTPUT_FILE_OBJECT);
	        		filePath = file.getAbsolutePath();
	        	}
	        }
	    	
	        String message = fileCreated? "File created" : "File opened";
	        message += ": " + filePath;
	    	Toast toast = Toast.makeText(AFileDialogTestingActivity.this, message, Toast.LENGTH_LONG);
			toast.show();
	    }
	}
```
	
Il faut tenir en compte que si un fichier a �t� cr��, alors le valeur (� l�int�rieur de l'objet _Bundle_) repr�sent� par la mot _FileChooserActivity.OUTPUT_NEW_FILE_NAME_ contiendra le nom de fichier et le valeur repr�sent� par la mot _FileChooserActivity.OUTPUT_FILE_OBJECT_ contiendra le dossier dans lequel le fichier doit �tre cr��. Autrement, si le fichier a �t� seulement s�lectionn�, _FileChooserActivity.OUTPUT_NEW_FILE_NAME_ contiendra un valeur nulle et _FileChooserActivity.OUTPUT_FILE_OBJECT_ contiendra le fichier s�lectionn�.

### 1.4 Demo

Dans le dossier [demo](../demo/) on peut trouver le projet d'une application r�f�rence et utilise la librairie _aFileDialog_.

On peut aussi installer cette application dans [Google Play](https://play.google.com/store/apps/details?id=ar.com.daidalos.afiledialog.test).

##  2. Options avanc�s 

###  2.1. S�lectionner dossiers 

Par d�faut, le s�lecteur s�lectionne fichiers, n�anmoins, on le peut param�trer pour s�lectionner dossiers. Dans ce cas, un bouton "Ok" sera affich� � la section inf�rieur du s�lecteur de fichiers; en appuyant ce bouton on s�lectionnera le dossier actuelle (�tant donn� que l��v�nement de toucher un dossier s'utilise pour ouvrir le dossier et afficher tous les fichiers que sont � l�int�rieur). 

Si on utilise un _Dialog_ pour afficher le s�lecteur de fichiers, alors il faut appeler le m�thode _setFolderMode(), de la classe _FileChooserDialog_, en utilisant "true" comme param�tre:

```java
    FileChooserDialog dialog = new FileChooserDialog(this);
    dialog.setFolderMode(true);
    dialog.show();
```
	
Si on utilise une _Activity_ pour afficher le s�lecteur de fichiers, alors on doit ajouter un extra au _Intent_ avec le nom _FileChooserActivity.INPUT_FOLDER_MODE_ et avec le valeur "true":

```java
    Intent intent = new Intent(this, FileChooserActivity.class);
    intent.putExtra(FileChooserActivity.INPUT_FOLDER_MODE, true);
    this.startActivityForResult(intent, 0);
```

###  2.2. Cr�er fichiers 

On peut param�trer le s�lecteur de fichiers pour afficher un bouton "New", pour donner la possibilit� aux utilisateurs de cr�er fichiers ou dossiers. En appuyant ce bouton, une fen�tre �mergent que permettra � l'utilisateur de saisir le nom du fichier ou dossier qu'il veut cr�er.

Il faut tenir en compte que le s�lecteur de fichiers ne cr�e pas les fichiers ou dossiers, il se limite � obtenir le dossiers, dans lequel le fichier doit �tre cr��, et le nom du fichier. Il est responsabilit� de l'application, que a ouvert le s�lecteur de fichiers, de cr�er le fichier et de l'ajouter l'extension correcte.

Si on utilise un _Dialog_, alors on doit appeler le m�thode _setCanCreateFiles()_ de la classe _FileChooserDialog_, en utilisant "true" comme param�tre:

```java
    FileChooserDialog dialog = new FileChooserDialog(this);
    dialog.setCanCreateFiles(true);
    dialog.show();
```
	
Si on utilise une _Activity_, alors on doit ajouter un extra dans le _Intent_ avec le nom _FileChooserActivity.INPUT_CAN_CREATE_FILES_ et le valeur "true":

```java
    Intent intent = new Intent(this, FileChooserActivity.class);
    intent.putExtra(FileChooserActivity.INPUT_CAN_CREATE_FILES, true);
    this.startActivityForResult(intent, 0);
```
	
###  2.3. Filtrer fichiers et dossiers

Par d�faut, le s�lecteur de fichiers permet � l'utilisateur de s�lectionner n'importe quel fichier, n�anmoins, on peut utiliser expressions r�guli�res pour d�finir quels fichiers peuvent �tre s�lectionn�, selon son nom ou extension.

Si on utilise un _Dialog_, alors on doit appeler le m�thode _setFilter()_ de la classe _FileChooserDialog_, en utilisant une expression r�guli�re comme param�tre (dans cet exemple, l'expression r�guli�re d�finit que seulement fichiers d'images peuvent �tre s�lectionn�):

```java
    FileChooserDialog dialog = new FileChooserDialog(this);
    dialog.setFilter(".*jpg|.*png|.*gif|.*JPG|.*PNG|.*GIF");
    dialog.show();
```
	
Si on utilise une _Activity_, alors on doit ajouter un extra au _Intent_ avec nom  _FileChooserActivity.INPUT_REGEX_FILTER_ et avec l'expression r�guli�re comme valeur:

```java
    Intent intent = new Intent(this, FileChooserActivity.class);
    intent.putExtra(FileChooserActivity.INPUT_REGEX_FILTER, ".*jpg|.*png|.*gif|.*JPG|.*PNG|.*GIF");
    this.startActivityForResult(intent, 0);
```

De mani�re analogue, on peut �galement filtrer les dossiers en appelant la m�thode _setFolderFilter () _, si vous utilisez un _Dialog_, ou en utilisant l'extra _FileChooserActivity.INPUT_REGEX_FOLDER_FILTER_, si vous utilisez une _Activity_.

> Il faut Noter que lors du filtrage de fichiers, l'expression r�guli�re est appliqu�e au nom du fichier, mais lors du filtrage des dossiers, l'expression r�guli�re est appliqu�e sur le chemin absolu du fichier.

Quand on filtre des fichiers ou dossiers, on peut d�finir aussi si les fichiers/dossiers que ne sont pas s�lectionnables doivent �tre affich�s ou cach�s (quand il sont affich�s, il sont montr�s avec un couleur grise, � fin d'indiquer � l'utilisateur que il ne le peut pas s�lectionner).

Pour faire �a, si on utilise un _Dialog_, alors on doit appeler le m�thode _setShowOnlySelectable()_ de la classe _FileChooserDialog_; si on utilise une _Activity_, alors on doit ajouter un extra au _Intent_ avec le nom _FileChooserActivity.INPUT_SHOW_ONLY_SELECTABLE_.

###  2.4. Afficher messages de confirmation 

Le s�lecteur de fichiers peut �tre param�tr�  pour afficher de fen�tres avec messages de confirmation (avec les options "Oui" et "Non") au s�lectionner ou cr�er un fichier, avec messages comme "Vous �tes sure que vous voulez ouvrir ce fichier?".

Si on utilise un _Dialog_, on doit appeler le m�thode _setShowConfirmation()_ de la classe _FileChooserDialog_, le premier param�tre d�finit si un fen�tre de confirmation doit �tre affich� au ouvrir un fichier et le deuxi�me param�tre si la fen�tre doit �tre affich� quand on cr�� un fichier:

```java
    FileChooserDialog dialog = new FileChooserDialog(this);
    dialog.setShowConfirmation(true, false);
    dialog.show();
```
	
Si on utilise une _Activity_, on doit ajouter deux extras au _Intent_ avec les noms _FileChooserActivity.INPUT_SHOW_CONFIRMATION_ON_CREATE_ et _FileChooserActivity.INPUT_SHOW_CONFIRMATION_ON_SELECT_:

```java
    Intent intent = new Intent(this, FileChooserActivity.class);
    intent.putExtra(FileChooserActivity.INPUT_SHOW_CONFIRMATION_ON_SELECT, true);
    intent.putExtra(FileChooserActivity.INPUT_SHOW_CONFIRMATION_ON_CREATE, false);
    this.startActivityForResult(intent, 0);
```
	
Il faut remarquer que les messages des fen�tres peuvent �tre chang�s avec la classe _FileChooserLabels_.

###  2.5. Param�trer �tiquettes 

On peut changer tous les messages et �tiquettes utilis�s par la librairie, pour ajouter de nouveaux messages ou pour ajouter des autre langages.

Pour faire �a, on doit cr�er une instance de la classe _FileChooserLabels_ et d�finir le valeur des textes qu'on veut modifier (dans le cas des messages de confirmation, le texte "$file_name" sera remplac� par le nom du fichier concern�):

```java
    FileChooserLabels labels = new FileChooserLabels();
    labels.createFileDialogAcceptButton = "Accept";
    labels.createFileDialogCancelButton = "Cancel";
    labels.createFileDialogMessage = "Enter the name of the file";
    labels.createFileDialogTitle = "Create file";
    labels.labelAddButton = "Add";
    labels.labelSelectButton = "Select";
    labels.messageConfirmCreation = "Are you sure that you want to create the file $file_name?";
    labels.messageConfirmSelection = "Are you sure that you want to open the file $file_name?";
    labels.labelConfirmYesButton = "Yes";
    labels.labelConfirmNoButton = "Note;
```
	
Apr�s, il faut passer cette instance au s�lecteur de fichiers. Si on utilise un _Dialog_, on doit appeler le m�thode _setLabels()_ de la classe _FileChooserDialog_:

```java
    FileChooserDialog dialog = new FileChooserDialog(this);
    dialog.setLabels(labels);
    dialog.show();
```
	
Si on utilise une _Activity_, on doit ajouter un extra au _Intent_ avec le nom _FileChooserActivity.INPUT_LABELS_:

```java
    Intent intent = new Intent(this, FileChooserActivity.class);
    intent.putExtra(FileChooserActivity.INPUT_LABELS, (Serializable) labels); 
    this.startActivityForResult(intent, 0);
```