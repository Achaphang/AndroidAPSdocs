# Χαρακτηριστικά του OpenAPS

## Autosens

* Autosens is a algorithm which looks at blood glucose deviations (positive/negative/neutral).
* It will try and figure out how sensitive/resistant you are based on these deviations.
* The oref implementation in **OpenAPS** runs off a combination of 24 and 8 hours worth of data. It uses either one which is more sensitive.
* In versions prior to AAPS 2.7 user had to choose between 8 or 24 hours manually.
* From AAPS 2.7 on Autosens in AAPS will switch between a 24 and 8 hours window for calculating sensitivity. It will pick which ever one is more sensitive. 
* If users have come from oref1 they will probably notice the system may be less dynamic to changes, due to the varying of either 24 or 8 hours of sensitivity.
* Changing a cannula or changing a profile will reset Autosens ratio back to 0%.
* Autosens adjusts your basal, I:C and ISF for you (i.e.: mimicking what a Profile shift does).
* If continuously eating carbs over an extended period, autosens will be less effective during that period as carbs are excluded from BG delta calculations.

## Super Micro Bolus (SMB)

Το SMB, το συντομογραφία του πολύ μικρό bolus «Super Micro Bolus», είναι το τελευταίο χαρακτηριστικό του OpenAPS (από το 2018) στο πλαίσιο του αλγορίθμου Oref1. Σε αντίθεση με το AMA, το SMB δεν χρησιμοποιεί προσωρινά βασικά ποσοστά για τον έλεγχο των επιπέδων γλυκόζης, αλλά κυρίως ** μικρά σούπερ μικροbolus **. Σε καταστάσεις όπου το AMA θα προσθέσει 1.0 IU ινσουλίνη χρησιμοποιώντας ένα προσωρινό βασικό ρυθμό, το SMB παράγει διάφορα σούπερ μικροbolus σε μικρά στάδια σε διαστήματα ** 5 λεπτών **, π.χ. 0,4 IU, 0,3 IU, 0,2 IU και 0,1 IU. Ταυτόχρονα (για λόγους ασφαλείας) ο πραγματικός βασικός ρυθμός ρυθμίζεται σε 0 IU / h για μια ορισμένη χρονική περίοδο για να αποφευχθεί η υπερβολική δόση (** «μηδενικός ρυθμός» **). Αυτό επιτρέπει στο σύστημα να ρυθμίζει τη γλυκόζη αίματος γρηγορότερα από ότι με την προσωρινή αύξηση βασικού ρυθμού στο AMA.

Χάρη στην SMB, μπορεί να είναι αρκετό για τα γεύματα με χαμηλή περιεκτικότητα σε υδατάνθρακες για να ενημερώσουν το σύστημα για την προγραμματισμένη ποσότητα υδατανθράκων και να αφήσουν τα υπόλοιπα στο AAPS. Ωστόσο, αυτό μπορεί να οδηγήσει σε υψηλότερες αιχμές postprandial, διότι δεν είναι δυνατή το προ-bolus. Ή εάν δώσετε, εάν χρειάζεται ένα προ-bolus, ένα ** bolus ξεκινήματος**, το οποίο ** μόνο εν μέρει ** καλύπτει τους υδατάνθρακες (π.χ. 2/3 του εκτιμώμενου ποσού) το υπόλοιπο καλύπτετε από το SMB.

Η λειτουργία SMB περιλαμβάνει κάποιους μηχανισμούς ασφαλείας:

1. Η μεγαλύτερη μοναδική δόση SMB μπορεί να είναι η μικρότερη μόνο τιμή των:
    
    * value corresponding to the current basal rate (as adjusted by autosens) for the duration set in "Max minutes of basal to limit SMB to", e.g. basal quantity for the next 30 minutes, or
    * το ήμισυ της απαιτούμενης ποσότητας ινσουλίνης, ή
    * το υπόλοιπο τμήμα της μέγιστης τιμής Iob σας στις ρυθμίσεις.

2. Πιθανώς συχνά θα παρατηρήσετε χαμηλά προσωρινά βασικά ποσοστά (αποκαλούμενα «χαμηλός ρυθμός») ή προσωρινά βασικά ποσοστά στα 0 U / h (αποκαλούμενα «μηδενικός ρυθμός»). Αυτό είναι σχεδιασμένο για λόγους ασφαλείας και δεν έχει αρνητικές επιπτώσεις αν το προφίλ έχει ρυθμιστεί σωστά. Η καμπύλη IOB έχει μεγαλύτερη σημασία από την πορεία των προσωρινών βασικών τιμών.

3. Πρόσθετοι υπολογισμοί για την πρόβλεψη της πορείας της γλυκόζης, π.χ. από UAM (μη αναγγελθέντα γεύματα). Ακόμα και χωρίς την εισαγωγή υδατανθράκων από τον χρήστη, το UAM μπορεί να ανιχνεύσει αυτόματα μια σημαντική αύξηση των επιπέδων γλυκόζης λόγω γευμάτων, αδρεναλίνης ή άλλων επιδράσεων και να προσπαθήσει να το προσαρμόσει με SMB. Για να είμαστε στην ασφαλή πλευρά, αυτό λειτουργεί και στην άλλη κατεύθυνση και μπορεί να σταματήσει την SMB νωρίτερα, εάν εμφανιστεί μια απροσδόκητα γρήγορη πτώση της γλυκόζης. Αυτός είναι ο λόγος για τον οποίο η UAM πρέπει να είναι πάντα ενεργή στην SMB.

**You must have started [objective 10](../Usage/Objectives#objective-10-enabling-additional-oref1-features-for-daytime-use-such-as-super-micro-bolus-smb) to use SMB.**

Δείτε επίσης: [ Τεκμηρίωση OpenAPS για oref1 SMB ](https://openaps.readthedocs.io/en/latest/docs/Customize-Iterate/oref1.html) και [ Πληροφορίες Tim για SMB ](http://www.diabettech.com/artificial-pancreas/understanding-smb-and-oref1/).

### Μέγιστη τιμή U / h μια τιμή βασικού ρυθμού μπορεί να ρυθμιστεί σε (OpenAPS "μέγιστος βασικός")

Αυτή η ρύθμιση ασφαλείας καθορίζει το μέγιστο προσωρινό βασικό ρυθμό που μπορεί να προσφέρει η αντλία ινσουλίνης. Η τιμή πρέπει να είναι η ίδια στην αντλία και στο AAPS και πρέπει να είναι τουλάχιστον 3 φορές υψηλότερη από τη μοναδική βασική τιμή.

Παράδειγμα:

Ο μέγιστος ρυθμός του βασικού προφίλ σας κατά τη διάρκεια της ημέρας είναι 1,00 U / h. Τότε συνιστάται μια μέγιστη βασική τιμή τουλάχιστον 3 U / h.

Αλλά δεν μπορείτε να επιλέξετε οποιαδήποτε αξία. Το AAPS περιορίζει την τιμή ως «αυστηρό όριο» ανάλογα με την ηλικία του ασθενή που έχετε επιλέξει κάτω από τις ρυθμίσεις. Η χαμηλότερη επιτρεπόμενη τιμή είναι για τα παιδιά και η υψηλότερη για ενήλικες ανθεκτικούς στην ινσουλίνη.

Το AndroidAPS περιορίζει την τιμή ως εξής:

* Child: 2
* Έφηβος: 5
* Adult: 10
* Insulin-resistant adult: 12

### Το μέγιστο συνολικό IOB που το OpenAPS δεν μπορεί να υπερβεί (OpenAPS "max-iob")

Αυτή η τιμή καθορίζει ποια μέγιστη IOB πρέπει να ληφθεί υπόψη από το AAPS που λειτουργεί σε λειτουργία κλειστού κυκλώματος. Εάν ο τρέχων IOB (π.χ. μετά από ένα bolus γεύματος) είναι πάνω από την καθορισμένη τιμή, το κύκλωμα σταματά να χορηγεί την ινσουλίνη έως ότου το όριο IOB είναι χαμηλότερο από τη δεδομένη τιμή.

Χρησιμοποιώντας το OpenAPS SMB, το μέγιστο IOB υπολογίζεται διαφορετικά από το OpenAPS AMA. Στην AMA, το μέγιστο IOB ήταν απλώς μια παράμετρος ασφαλείας για το βασικό IOB, ενώ σε λειτουργία SMB, περιλαμβάνει επίσης το bolus IOB. Ένα καλό ξεκίνημα είναι

    μέγιστο IOB = μέσος bolus γεύματος + 3 φορές το μέγιστο ημερησίως βασικό
    

Να είστε προσεκτικοί και υπομονετικοί και μόνο να αλλάζετε τις ρυθμίσεις βήμα προς βήμα. Είναι διαφορετικό για όλους και επίσης εξαρτάται από τη μέση συνολική ημερήσια δόση (TDD). Για λόγους ασφάλειας, υπάρχει ένα όριο, το οποίο εξαρτάται από την ηλικία του ασθενούς. Το «αυστηρό όριο» για το μέγιστο IOB είναι υψηλότερο από το AMA.

* Child: 3
* Teenage: 7
* Adult: 12
* Insulin resistant adult: 25

Δείτε επίσης [ την τεκμηρίωση OpenAPS για SMB ](https://openaps.readthedocs.io/en/latest/docs/Customize-Iterate/oref1.html#understanding-smb).

### Ενεργοποιήστε το AMA Autosense

Εδώ, μπορείτε να επιλέξετε αν θέλετε να χρησιμοποιήσετε ή όχι την [ ανίχνευση ευαισθησίας ](../Configuration/Sensitivity-detection-and-COB.md) 'autosense'.

### Ενεργοποίησε το SMB

Εδώ μπορείτε να ενεργοποιήσετε ή να απενεργοποιήσετε πλήρως τη λειτουργία SMB.

### Ενεργοποίηση SMB με COB

Το SMB λειτουργεί όταν υπάρχει COB ενεργό.

### Ενεργοποιήστε τη λειτουργία SMB με τους προσωρινούς στόχους

Το SMB λειτουργεί όταν υπάρχει χαμηλός ή υψηλός προσωρινός στόχος ενεργός (πρόωρο γεύμα, δραστηριότητα, υπογλυκαιμία)

### Ενεργοποίηση SMB με υψηλούς προσωρινούς στόχους

Το SMB λειτουργεί όταν υπάρχει ένας υψηλός προσωρινός στόχος ενεργός (δραστηριότητα, υπογλυκαιμία). Αυτή η επιλογή μπορεί να περιορίσει άλλες ρυθμίσεις SMB, δηλαδή εάν είναι ενεργοποιημένη η επιλογή "SMB με υψηλούς προσωρινούς στόχους" και το στοιχείο "SMB με υψηλούς " απενεργοποιείται, το SMB λειτουργεί μόνο με χαμηλούς και όχι με υψηλούς προσωρινούς στόχους. Το ίδιο ισχύει και για την ενεργοποιημένη SMB με COB: αν είναι απενεργοποιημένη η επιλογή "SMB με ", τότε δεν υπάρχει SMB με υψηλό προσωρινό στόχο, ακόμη και αν είναι ενεργό το COB.

### Ενεργοποιήστε το SMB

Η SMB εργάζεται πάντοτε (ανεξάρτητα από το COB, τους προσωρινούς στόχους ή τα bolus). Για λόγους ασφαλείας, αυτή η επιλογή είναι πιθανώς για πηγές BG με ένα ωραίο σύστημα φιλτραρίσματος για θορυβώδη δεδομένα. Προς το παρόν, λειτουργεί μόνο με Dexcom G5, αν χρησιμοποιείτε την εφαρμογή Dexcom (patched) ή το "native mode" στο xDrip +. Εάν μια τιμή BG έχει πολύ μεγάλη απόκλιση, το G5 δεν το στέλνει και περιμένει την επόμενη τιμή σε 5 λεπτά.

Για άλλα CGM / FGM όπως το Freestyle Libre, το 'SMB πάντα' απενεργοποιείται μέχρι το xDrip + να έχει ένα καλύτερο plugin εξομάλυνσης θορύβου. Μπορείτε να βρείτε περισσότερα [ εδώ ](../Usage/Smoothing-Blood-Glucose-Data-in-xDrip.md).

### Ενεργοποιήστε το SMB μετά από τους υδατάνθρακες

Η SMB εργάζεται για 6 ώρες μετά τους υδατάνθρακες, ακόμη και αν η COB βρίσκεται στο 0. Για λόγους ασφαλείας, αυτή η επιλογή είναι πιθανώς για πηγές BG με ένα ωραίο σύστημα φιλτραρίσματος για θορυβώδη δεδομένα. Προς το παρόν, λειτουργεί μόνο με Dexcom G5, αν χρησιμοποιείτε την εφαρμογή Dexcom (patched) ή το "native mode" στο xDrip +. Εάν μια τιμή BG έχει πολύ μεγάλη απόκλιση, το G5 δεν το στέλνει και περιμένει την επόμενη τιμή σε 5 λεπτά.

Για άλλα CGM / FGM όπως το Freestyle Libre, το "SMB πάντα" απενεργοποιείται μέχρι το xDrip + να έχει ένα καλύτερο plugin εξομάλυνσης θορύβου. Μπορείτε να βρείτε [ περισσότερες πληροφορίες εδώ ](../Usage/Smoothing-Blood-Glucose-Data-in-xDrip.md).

### Τα μέγιστα λεπτά του βασικού ρυθμού που περιορίζουν το SMB

Αυτή είναι μια σημαντική ρύθμιση ασφάλειας. This value determines how much SMB can be given based on the amount of basal insulin in a given time, when it is covered by COBs.

Αυτό κάνει το SMB πιο επιθετικό. Για αρχή, θα πρέπει να ξεκινήσετε με την προεπιλεγμένη τιμή των 30 λεπτών. Μετά από κάποια εμπειρία, μπορείτε να αυξήσετε την αξία με βήματα 15 λεπτών και να παρακολουθήσετε τον τρόπο με τον οποίο επηρεάζουν αυτές οι αλλαγές.

Συνιστάται να μην ορίσετε την τιμή μεγαλύτερη από 90 λεπτά, καθώς αυτό θα οδηγούσε σε ένα σημείο όπου ο αλγόριθμος μπορεί να μην είναι σε θέση να ρυθμίσει μια φθίνουσα BG με 0 IE / h βασικό ('μηδενικό ρυθμό'). Θα πρέπει επίσης να ρυθμίσετε τους συναγερμούς, ειδικά αν δοκιμάζετε ακόμα νέες ρυθμίσεις, οι οποίες σας προειδοποιούν πριν προχωρήσετε σε υπογλυκαιμίες.

Προεπιλεγμένη τιμή: 30 λεπτά.

### Ενεργοποίηση UAM

Με αυτήν την επιλογή ενεργοποιημένη, ο αλγόριθμος SMB μπορεί να αναγνωρίσει τα μη εισαγόμενα γεύματα. Αυτό είναι χρήσιμο αν ξεχάσετε να ενημερώσετε το AndroidAPS σχετικά με τους υδατάνθρακες σας ή να εκτιμήσετε εσφαλμένα τους υδατάνθρακες σας και η ποσότητα των εισαγόμενων υδατανθράκων είναι λάθος ή εάν ένα γεύμα με πολλά λιπαρά και πρωτεΐνες έχει μεγαλύτερη διάρκεια από την αναμενόμενη. Χωρίς εισαγωγή υδατανθράκων, το UAM μπορεί να αναγνωρίσει γρήγορες αυξήσεις της γλυκόζης που προκαλούνται από υδατάνθρακες, αδρεναλίνη κ. λπ. και προσπαθεί να το προσαρμόσει με τις SMBs. Αυτό συμβαίνει και με τον αντίθετο τρόπο: εάν υπάρξει γρήγορη μείωση της γλυκόζης, μπορεί να σταματήσει τα SMBs νωρίτερα.

**Επομένως, η UAM θα ​​πρέπει πάντα να ενεργοποιείται όταν χρησιμοποιείται SMB.**

### Ο υψηλός προσωρινός στόχος ανεβάζει την ευαισθησία

Εάν έχετε ενεργοποιήσει αυτήν την επιλογή, η ευαισθησία στην ινσουλίνη θα αυξηθεί ενώ θα έχετε προσωρινό στόχο πάνω από 100 mg / dl ή 5,6 mmol / l. Αυτό σημαίνει ότι η ISF θα αυξηθεί, ενώ η IC και ο βασικός θα μειωθούν.

### Ο χαμηλός προσωρινός στόχος μειώνει την ευαισθησία

Εάν έχετε ενεργοποιήσει αυτήν την επιλογή, η ευαισθησία στην ινσουλίνη θα μειωθεί ενώ θα έχετε προσωρινό στόχο κάτω από 100 mg / dl ή 5,6 mmol / l. Αυτό σημαίνει ότι η ISF θα μειωθεί και θα αυξηθεί το IC και η βασική.

### Προηγμένες ρυθμίσεις

** Χρησιμοποιείτε πάντα το σύντομο μέσο δέλτα αντί για απλά δεδομένα. **Εάν ενεργοποιήσετε αυτή τη λειτουργία, το AndroidAPS χρησιμοποιεί το σύντομο μέσο όρο γλυκόζης δέλτα / αίματος από τα τελευταία 15 λεπτά, το οποίο είναι συνήθως ο μέσος όρος των τριών τελευταίων τιμών. Αυτό βοηθά το AndroidAPS να λειτουργεί πιο σταθερά με θορυβώδεις πηγές δεδομένων όπως το xDrip + και το Libre.

** Μέγιστος καθημερινός πολλαπλασιαστής ασφαλείας ** Αυτό είναι ένα σημαντικό όριο ασφάλειας. Η προεπιλεγμένη ρύθμιση (η οποία είναι απίθανο να χρειαστεί ρύθμιση) είναι 3. Αυτό σημαίνει ότι το AndroidAPS δεν θα επιτρέπεται ποτέ να ρυθμίσει ένας προσωρινός βασικός ρυθμός που υπερβαίνει το 3x του υψηλότερου ωριαίου βασικού ρυθμού που έχει προγραμματιστεί στην αντλία ενός χρήστη. Παράδειγμα: Εάν ο υψηλότερος βασικός σας ρυθμός είναι 1,0 U / h και ο μέγιστος ημερήσιος πολλαπλασιαστής ασφαλείας είναι 3, τότε το AndroidAPS μπορεί να θέσει ένα μέγιστο προσωρινό βασικό ρυθμό 3,0 U / h (= 3 x 1,0 U / h).

Προεπιλεγμένη τιμή: 3 (δεν πρέπει να αλλάξει αν δεν χρειάζεται πραγματικά και ξέρετε και τι κάνετε)

** Πολλαπλασιαστής ασφάλειας βασικού ** Αυτό είναι ένα άλλο σημαντικό όριο ασφάλειας. Η προεπιλεγμένη ρύθμιση (η οποία είναι επίσης απίθανο να χρειαστεί ρύθμιση) είναι 4. Αυτό σημαίνει ότι το AndroidAPS δεν θα επιτρέπεται ποτέ να ορίσει ένα προσωρινό βασικό ρυθμό που υπερβαίνει το 4x την τρέχουσα ωριαία βασική τιμή που έχει προγραμματιστεί στην αντλία του χρήστη.

Προεπιλεγμένη τιμή: 4 (δεν πρέπει να αλλάξει, εκτός αν πραγματικά χρειάζεται και ξέρεις τι κάνεις)

* * *

## Advanced Meal Assist (AMA)

AMA, η σύντομη μορφή του "advanced meal assist" είναι μια δυνατότητα OpenAPS από το 2017 (oref0). Το OpenAPS Advanced Meal Assist (Προηγμένος Βοηθός Γεύματος) (AMA) επιτρέπει στο σύστημα να φτάσει σε υψηλούς ρυθμούς πιο γρήγορα μετά από ένα bolus γεύματος, αν εισάγετε αξιόπιστα τους υδατάνθρακες.

You can find more information in the [OpenAPS documentation](http://openaps.readthedocs.io/en/latest/docs/walkthrough/phase-4/advanced-features.html#advanced-meal-assist-or-ama).

### Μέγιστη τιμή U / h μια τιμή βασικού ρυθμού μπορεί να ρυθμιστεί σε (OpenAPS "μέγιστος βασικός")

This safety setting helps AndroidAPS from ever being capable of giving a dangerously high basal rate and limits the temp basal rate to x U/h. It is advised to set this to something sensible. A good recommendation is to take the highest basal rate in your profile and multiply it by 4 and at least 3. For example, if the highest basal rate in your profile is 1.0 U/h you could multiply that by 4 to get a value of 4 U/h and set the 4 as your safety parameter.

You cannot chose any value: For safety reason, there is a 'hard limit', which depends on the patient age. The 'hard limit' for maxIOB is lower in AMA than in SMB. For children, the value is the lowest while for insulin resistant adults, it is the biggest.

The hardcoded parameters in AndroidAPS are:

* Child: 2
* Έφηβος: 5
* Adult: 10
* Ανθεκτικός στην ινσουλίνη ενήλικος: 12

### Το μέγιστο συνολικό IOB που το OpenAPS μπορεί να δώσει\[U\] (OpenAPS "max-iob")

This parameter limits the maximum of basal IOB where AndroidAPS still works. If the IOB is higher, it stops giving additional basal insulin until the basal IOB is under the limit.

The default value is 2, but you should be rise this parameter slowly to see how much it affects you and which value fits best. Είναι διαφορετικό για όλους και επίσης εξαρτάται από τη μέση συνολική ημερήσια δόση (TDD). Για λόγους ασφάλειας, υπάρχει ένα όριο, το οποίο εξαρτάται από την ηλικία του ασθενούς. The 'hard limit' for maxIOB is lower in AMA than in SMB.

* Child: 3
* Έφηβος: 5
* Adult: 7
* Ανθεκτικός στην ινσουλίνη ενήλικος: 12

### Ενεργοποιήστε το AMA Autosense

Here, you can chose, if you want to use the [sensitivity detection](../Configuration/Sensitivity-detection-and-COB.md) autosense or not.

### Το autosens ρυθμίζει επίσης προσωρινούς στόχους

If you have this option enabled, autosense can adjust targets (next to basal, ISF and IC), too. This lets AndroidAPS work more 'aggressive' or not. The actual target might be reached faster with this.

### Προηγμένες ρυθμίσεις

** Χρησιμοποιείτε πάντα το σύντομο μέσο δέλτα αντί για απλά δεδομένα. **Εάν ενεργοποιήσετε αυτή τη λειτουργία, το AndroidAPS χρησιμοποιεί το σύντομο μέσο όρο γλυκόζης δέλτα / αίματος από τα τελευταία 15 λεπτά, το οποίο είναι συνήθως ο μέσος όρος των τριών τελευταίων τιμών. Αυτό βοηθά το AndroidAPS να λειτουργεί πιο σταθερά με θορυβώδεις πηγές δεδομένων όπως το xDrip + και το Libre.

** Μέγιστος καθημερινός πολλαπλασιαστής ασφαλείας ** Αυτό είναι ένα σημαντικό όριο ασφάλειας. Η προεπιλεγμένη ρύθμιση (η οποία είναι απίθανο να χρειαστεί ρύθμιση) είναι 3. Αυτό σημαίνει ότι το AndroidAPS δεν θα επιτρέπεται ποτέ να ρυθμίσει ένας προσωρινός βασικός ρυθμός που υπερβαίνει το 3x του υψηλότερου ωριαίου βασικού ρυθμού που έχει προγραμματιστεί στην αντλία ενός χρήστη. Παράδειγμα: Εάν ο υψηλότερος βασικός σας ρυθμός είναι 1,0 U / h και ο μέγιστος ημερήσιος πολλαπλασιαστής ασφαλείας είναι 3, τότε το AndroidAPS μπορεί να θέσει ένα μέγιστο προσωρινό βασικό ρυθμό 3,0 U / h (= 3 x 1,0 U / h).

Προεπιλεγμένη τιμή: 3 (δεν πρέπει να αλλάξει αν δεν χρειάζεται πραγματικά και ξέρετε και τι κάνετε)

** Πολλαπλασιαστής ασφάλειας βασικού ** Αυτό είναι ένα άλλο σημαντικό όριο ασφάλειας. Η προεπιλεγμένη ρύθμιση (η οποία είναι επίσης απίθανο να χρειαστεί ρύθμιση) είναι 4. Αυτό σημαίνει ότι το AndroidAPS δεν θα επιτρέπεται ποτέ να ορίσει ένα προσωρινό βασικό ρυθμό που υπερβαίνει το 4x την τρέχουσα ωριαία βασική τιμή που έχει προγραμματιστεί στην αντλία του χρήστη.

Προεπιλεγμένη τιμή: 4 (δεν πρέπει να αλλάξει, εκτός αν πραγματικά χρειάζεται και ξέρεις τι κάνεις)

**Bolus snooze dia divisor** The feature “bolus snooze” works after a meal bolus. AAPS doesn’t set low temporary basal rates after a meal in the period of the DIA divided by the “bolus snooze”-parameter. The default value is 2. That means with a DIA of 5h, the “bolus snooze” would be 5h : 2 = 2.5h long.

Default value: 2