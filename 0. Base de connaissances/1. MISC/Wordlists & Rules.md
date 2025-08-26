
- [[#Générer une wordlist]]
- [[#Reformatter wordlist]]
- [[#Rules]]

---
# Générer une wordlist

- [[#Username Anarchy]]
- [[#CUPP]]
- [[#CRUNCH]]
- [[#PYDICTOR]]
- [[#SEQ]]
- [[#HASHCAT]]
- [[#REPOLIST]]

### CEWL

Wordlist en extrayant les données d'une page web

```bash
cewl example.com --min_word_length 5 --depth 3 --write words.txt
```

### Username Anarchy

https://github.com/urbanadventurer/username-anarchy.git

```bash
./username-anarchy  <prenom> <nom> > <file.txt>
```

A partir d'un fichier

```shell
./username-anarchy -i /home/ltnbob/names.txt > usernames
```

en precisant un format

```shell
./username-anarchy --list-formats
```

```shell
./username-anarchy -i ./names.txt  --select-format first.last > usernames
```

### CUPP

Mot de passe personnalisés avec des infos sur quelqu'un (judicieux de commencer avec peu d'informations)

```bash
cupp -i
```

### CRUNCH

8 caractères commencant par Ab suivi par 6 digits.

```shell-session
crunch 8 8 0123456789 -t Ab%%%%%% -o number_passwords.txt
```

### PYDICTOR

https://github.com/LandGrey/pydictor

Aaaa0000

```bash
python3 pydictor.py -pattern "[A-G]{1,1}<none>[a-z]{1,1}<none>[a-z]{1,1}<none>[a-z]{1,1}<none>[0-9]{1,1}<none>[0-9]{1,1}<none>[0-9]{1,1}<none>[0-9]{1,1}<none>" -o wordlistdeouf.txt

```

### SEQ

Génère une wordlist de 0000 à 9999

```bash
seq -w 0 9999 > tokens.txt
```

### HASHCAT

Mutated wordlist avec une wordlist et un fichier de règle

```bash
hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```


**bons fichiers de règles:**

best64 -> /usr/share/hashcat/rules/best64.rule

**Fichiers de règles ultime :**
- https://github.com/NotSoSecure/password_cracking_rules
- https://github.com/rarecoil/pantagrule


### REPOLIST

Générer wordlist à partir d'un repo git

https://github.com/Ademking/repolist

```
repolist -u https://gihtub.com/user/repo
```




---

# Reformatter wordlist

### Supprimer les mots faisant moins de 7 caractères (inférieur ou égal à 6)

SED

```bash
cat file.txt | sed -r '/^.{,6}$/d' > seven_plus_char.txt
```

GREP

```bash
cat file.txt | grep -E '.{8}' > seven_plus_char.txt
```

### Supprimer les mots plus de 7 caractères (supérieur ou égal à 8)

```bash
cat file.txt | sed -r '/^.{8,}$/d' > seven_minus_char.txt
```
### Supprimer les mots n'ayant pas de caractères spéciaux

SED

```bash
cat file.txt | sed -r '/[!-/:-@\[-`\{-~]+/!d' > special_char.txt
```

### Supprimer les mots n'ayant pas de chiffres

SED

```bash
cat file.txt | sed -r '/[0-9]+/!d' > Number.txt
```

GREP

```BASH
grep '[[:digit:]]'
```
### Supprimer les mots n'ayant pas de minuscules

SED

```bash
cat file.txt | sed -r '/[a-z]+/!d' > Maj.txt
```

GREP

```BASH
grep '[[:lower:]]'
```
### Supprimer les mots n'ayant pas de majuscules

SED

```bash
cat file.txt | sed -r '/[A-Z]+/!d' > Min.txt
```

GREP

```bash
grep '[[:upper:]]'
```

### Récupérer les mots commençants par A

```bash
grep -oE '\bA\S*' password.list > passwordA.list 
```

---
# Rules

default rules

```shell
ls -l /usr/share/hashcat/rules/
```

Fichiers de règles ultime :
- https://github.com/NotSoSecure/password_cracking_rules
- https://github.com/stealthsploit/OneRuleToRuleThemStill
- https://github.com/rarecoil/pantagrule

| **Rule**                    | **Description**                                | **Link**                                                                                              |
| --------------------------- | ---------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `one-rule-to-rule-them-all` | One super rule that tops all the tests         | [Link](https://github.com/stealthsploit/OneRuleToRuleThemStill/blob/main/OneRuleToRuleThemStill.rule) |
| `rules-collection`          | Largest collection of hashcat rule-files       | [Link](https://github.com/n0kovo/hashcat-rules-collection)                                            |
| `nsa-rules`                 | Rules generated from cracked passwords         | [Link](https://github.com/NSAKEY/nsa-rules)                                                           |
| `Hob0Rules`                 | Rule based on statistics and industry patterns | [Link](https://github.com/praetorian-inc/Hob0Rules)                                                   |


## Générer un fichier de règles

```shell
echo 'c so0 si1 se3 ss5 sa@ $2 $0 $1 $9' > rule.txt
```

| **Function**    | **Description**                                                    | **Input**                             | **Output**                                                                                                        |
| --------------- | ------------------------------------------------------------------ | ------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| l               | Convert all letters to lowercase                                   | InlaneFreight2020                     | inlanefreight2020                                                                                                 |
| u               | Convert all letters to uppercase                                   | InlaneFreight2020                     | INLANEFREIGHT2020                                                                                                 |
| c / C           | capitalize / lowercase first letter and invert the rest            | inlaneFreight2020 / Inlanefreight2020 | Inlanefreight2020 / iNLANEFREIGHT2020                                                                             |
| t / TN          | Toggle case : whole word / at position N                           | InlaneFreight2020                     | iNLANEfREIGHT2020                                                                                                 |
| d / q / zN / ZN | Duplicate word / all characters / first character / last character | InlaneFreight2020                     | InlaneFreight2020InlaneFreight2020 / IInnllaanneeFFrreeiigghhtt22002200 / IInlaneFreight2020 / InlaneFreight20200 |
| { / }           | Rotate word left / right                                           | InlaneFreight2020                     | nlaneFreight2020I / 0InlaneFreight202                                                                             |
| ^X / $X         | Prepend / Append character X                                       | InlaneFreight2020 (^! / $! )          | !InlaneFreight2020 / InlaneFreight2020!                                                                           |
| r               | Reverse                                                            | InlaneFreight2020                     | 0202thgierFenalnI                                                                                                 |
| s               | Substitute                                                         | InlaneFreight2020 (se3)               | Inlan3Fr3ight2020                                                                                                 |
|                 |                                                                    |                                       |                                                                                                                   |

