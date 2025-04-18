# SNAP guidelines
This repository contains a list of guidelines to use Sentinel-1 SLC assets inside SNAP to create different products:
1) interferograms;
2) Digital Elevation Models (DEMs).
# Interferograms
Based on the tutorials:
- on creating the interferogram of [NASA](https://www.earthdata.nasa.gov/learn/data-recipes/create-interferogram-using-esas-sentinel-1-toolbox), [Skywatch](https://step.esa.int/docs/tutorials/S1TBX%20TOPSAR%20Interferometry%20with%20Sentinel-1%20Tutorial_v2.pdf), [ESA](https://esamultimedia.esa.int/multimedia/publications/TM-19/TM-19_InSAR_web.pdf);
- on phase-unwrapping of [NASA](https://www.earthdata.nasa.gov/learn/data-recipes/phase-unwrap-interferogram)

Follow this steps:
1) download two Sentinel-1 SLC images as *zip* files, one before and one after the date of earthquake or landslide. There are many sources for Sentinel-1 data. I suggest you to try with [Earthdata NASA](https://search.earthdata.nasa.gov) and [Copernicus Dataspace](https://browser.dataspace.copernicus.eu/). Both websites require free registration;
![image](https://github.com/user-attachments/assets/65aa0fc5-589d-4cfe-ab39-4e87621e400d) ![image](https://github.com/user-attachments/assets/f6631028-e2ef-4c11-891c-cfdfddd34dd5)
2) SAR instruments have different configurations and expert can give you advice on which data is better to choose to create proper interferograms. I am here to suggest to choose that images before and after should be from the same configuration (IW,SM,EW,WV,ascending,descending) but can be from different instrument (S1A and S1B);
3) open the SLC layers on SNAP. As a check, try to double click on an amplitude image and look at the result on the right panel;
![image](https://github.com/user-attachments/assets/802808a7-10c4-4b5c-9494-2017860739ad) ![image](https://github.com/user-attachments/assets/22c84d90-70f8-4455-89a6-d44743b1ef28)
4) *_Orb_Stack* coregistration of images. The tools is called *S-1 TOPS coregistration* and can be done with or without ESD (bursts-splitted images). Select in the *TOPSAR-Split* tab a sub-swath of the full image to coregister the images. If your S1 images have more than one polarization here you can choose which band to use for coregistration. In the Apply-Orbit-File tab you can choose orbital corrections: the default *Sentinel Precise* will choose the effemerides. Co-registration can also be assisted by DEM in *Back-Geocoding* tab. You can choose the default SRTM 3Sec or SRTM 1sec HGT. The *Output Deramp and Demod Phase* flag will try to improve the coregistration. After co-registration you will obtain a new object called *_Orb_Stack*. 
![image](https://github.com/user-attachments/assets/b5213dcd-c8d9-4bc4-9eb1-91be41a43249)

dopo aver finito la coregistrazione conviene visualizzare se il risultato è quanto sperato. Cliccando sull’immagine _Orb_Stack usiamo la funzione Open RGB Image Window e mettiamo l’immagine master sui canali R e G e l’immagine slave sul canale B. La coregistrazione è andata a buon fine se non vedo prevalenza di un colore blu rispetto al giallo e se i bordi sono netti; (_Orb_Stack_ifg) creazione dell’interferogramma moltiplicando un’immagine per la complessa coniugata dell’altra. Usando Interferogram Formation, effettuo tale operazione. Poiché siamo interessati ad isolare nella differenza di fase quella dovuta solamente allo spostamento del terreno, cerchiamo di togliere gli altri contributi alla differenza di fase. Inizio con flat-earth phase, ovvero rimuovo il contributo alla differenza di fase dovuto alla curvatura della Terra usando i dati orbitali e di curvatura dell’ellissoide. In questo passaggio creiamo anche un layer di coherence che serve per indicare quanto effettivamente è possibile distinguere una zona dove non c’è stata variazione ad una zona dove c’è stata variazione e sarà utile l’unwrapping. Nelle zone di bassa coerenza (<0.3) come le foreste, l’interferogramma non uscirà bene, mentre nelle zone di alta coerenza (>0.6) come le città l’interferogramma uscirà meglio; (_Orb_Stack_ifg_deb) per eliminare le strisce di separazione tra i burst, uso il TOPS deburst; (_Orb_Stack_ifg_deb_dinsar) voglio rimuovere ora anche il contributo topografico alla fase. Usando “Topographic phase removal” vado ad eliminare la topographic phase, cioè il contributo alla differenza di fase dovuto alla topografia. Se il mio obiettivo è ottenere un DEM non vado a rimuovere questo contributo perché è quello che mi serve. Se invece voglio ottenere un interferogramma allora mi baso su un DEM per sottrarre questo contributo alla fase; (_Orb_Stack_ifg_deb_ML_flt) rimangono altri contributi alla fase dovuti al rumore atmosferico, allo scattering di volume, alla variazione temporale degli singoli scatterers e al cambio di vista (quando uso immagini con angoli di vista differenti o se una è ascending e l’altra è descending). Per rimuovere questi contributi si usano delle operazioni: (_Orb_Stack_ifg_deb_ML) multi-looking: ovvero un filtro spaziale bidimensionale (come un kernel di convoluzione). Scegliamo le bande i (parte reale), q (parte immaginaria) e coh (coherence) su cui usare il multi-looking. In questo passaggio scompare la banda di fase perché è solo una banda virtuale derivata da i e q ma poi verrà fatta ricomparire dopo il prossimo filtro. Usiamo ad esempio un numero di look pari a 6; (_Orb_Stack_ifg_deb_ML_flt) Goldstein phase filtering: usa una FFT per aumentare il rapporto segnale-su-rumore. (_Orb_Stack_ifg_deb_ML_flt_subset) selezioniamo un subset dopo aver cliccato con il tasto destro sulla zona interessata e usato la funzione “Spatial subset from View”;
(_Orb_Stack_ifg_deb_ML_flt_subset_unw) ora eseguiamo l’unwrapping ovvero vogliamo avere una variazione continua della fase che non ricominci ogni 2π: si divide in tre fasi: esportiamo la wrapped phase con “Snaphu export”: selezionare 200 per row overlap e per column overlap. Qui si possono selezionare i parametri per l’unwrapping; usiamo “Snaphu unwrapping”: seleziono il prodotto che è stato scelto da esportare e come cartella seleziono la cartella del prodotto esportato. Uso il Display execution output per vedere se la procedura funziona. Se non funziona, bisogna andare nelle impostazioni “Manage External Tools”, selezionare Snaphu unwrapping e modificare in “Edit the selected operator”. Nella variabile USERPROFILE che si trova in System variables inserire la cartella del prodotto esportato. Far partire nuovamente lo strumento. Se non dovesse funzionare controllare di avere abbastanza spazio su disco; usiamo “Snaphu Import”: in ReadPhase seleziono il prodotto che è stato scelto da esportare. In Read-Unwrapped-Phase seleziono il file .hdr della unwrapped phase. In Snaphu-Import lascio non selezionato il non-salvataggio. In Write aggiungo _unw al nome. (_Orb_Stack_ifg_deb_ML_flt_subset_unw_dsp) vogliamo ora passare dalla fase unwrapped allo spostamento. Per questo usiamo “Phase to Displacement”. Quando si effettua questo passaggio viene lasciato indietro il layer di coherence. Se si vuole riportarlo si può usare Band Maths sul layer di displacement, tolgo la selezione Virtuale e come espressione preso il file _unw e seleziono la banda coherence; (_Orb_Stack_ifg_deb_ML_flt_subset_unw_dsp_TC) applico la funzione “Range Doppler Terrain correction” per riportare dalle slant range coordinates (che sono le coordinate radar) alle ground range coordinates (che sono le coordinate sul terreno). Per far questo si usa un DEM di riferimento. Se si vuole esportare su Google Earth conviene usare WGS84; esportare con “View as Google Earth KMZ” come file KMZ in modo da poterlo caricare su Google Earth.
# DEM generation
Based on the tutorials:
- [ESA](https://step.esa.int/docs/tutorials/S1TBX%20DEM%20generation%20with%20Sentinel-1%20IW%20Tutorial.pdf)
