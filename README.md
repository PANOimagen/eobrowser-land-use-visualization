# Land use visualization for Sentinel 2 using Linear Discriminant Analysis

Authors: Marta Elvira, Roberto Calvo, Javier Becerra. **[PANOimagen S.L.](http://www.panoimagen.com)**

This repository contains a script for EO Browser, specifically designed to visualize Sentinel-2 13 band data in a way that facilitates differentiation of urban areas (red channel), vegetation areas (green channel) and water areas (blue channel). The coefficients used to combine the original 13 bands for each channel have been obtained using three LDA classifiers (i.e. one per channel).


- Show [script](https://github.com/PANOimagen/eobrowser-land-use-visualization/blob/master/script.js).
- See in [EO Browser](https://apps.sentinel-hub.com/eo-browser/?lat=40.3840&lng=-3.6440&zoom=11&time=2019-09-26&preset=CUSTOM&datasource=Sentinel-2%20L1C&layers=B01,B02,B03&evalscript=cmV0dXJuIFtNYXRoLmFicyggMC40ODA5ICogKCgoKEIwMiAqIDIuNSkgLSAwLjMzMjkpICogLTkuNDQyNSkgKyAoKChCMDMgKiAyLjUpIC0gMC4zMTgyKSAqIDIuMTg0NikgKyAoKChCMDQgKiAyLjUpIC0gMC4zMzgwKSAqIDIuNTMzMykgKyAoKChCMTEgKiAyLjUpIC0gMC41NjQ0KSAqIDkuOTI1NikgKyAoKChCMTIgKiAyLjUpIC0gMC40MjE2KSAqIC0xMy42OTExKSkgKyAtMC40NzY2KSwKICAgICAgICBNYXRoLmFicyggMC4zMjc1ICogKCgoKEIwMiAqIDIuNSkgLSAwLjI4NDQpICogMTMuMzY0NCkgKyAoKChCMDMgKiAyLjUpIC0gMC4yNzM2KSAqIC02LjY1ODgpICsgKCgoQjA0ICogMi41KSAtIDAuMjc1MCkgKiAtMS4xOTk0KSArICgoKEIwOCAqIDIuNSkgLSAwLjU5NzIpICogLTAuMjA5MCkgKyAoKChCOEEgKiAyLjUpIC0gMC42NjQ4KSAqIDUuMTYzMCkgKyAoKChCMTEgKiAyLjUpIC0gMC41NjUxKSAqIC03LjIxODMpKSArIDAuMDQ2MyksCiAgICAgICAgTWF0aC5hYnMoIDAuMjM2MSAqICgoKChCMDMgKiAyLjUpIC0gMC4yNDI5KSAqIDIxLjg3NTkpICsgKCgoQjA0ICogMi41KSAtIDAuMjMyMSkgKiAtNi4wNjc5KSArICgoKEIwOCAqIDIuNSkgLSAwLjQzNzEpICogLTMuMDYwOCkgKyAoKChCMTEgKiAyLjUpIC0gMC40MTQ2KSAqIC00LjQ0MjApKSArIDAuMjA2MSldOw%3D%3D).

## General description

Principal Component Analysis (PCA) and Linear Discriminant Analysis (LDA) are two machine learning techniques commonly used to reduce the dimensionality of large data sets and variables, the result of both techniques being a linear combination of the data. PCA is used to decompose a multivariate dataset in a set of successive orthogonal components that explain a maximum amount of the variance in the data. LDA is a supervised version of PCA, which maximizes the separation between given classes. 

We have used LDA to create a visualization where each image channel (red, green and blue) codes the maximum information to identify respectively urban, crop and water related classes. Input class labels where taken from Spanish SIOSE land use classification. We have thus created three different transformations using LDA, one per component. Finally, as we have used a multiclass classifier for each type of data (for instance, urban data is separated into non urban, urban, industrial...), we transform the new axis obtained in order to translate the center of `non-urban` (resp. `non-vegetation` and `non-water`) classes to 0, and get the absolute value to recover any of the urban (resp. vegetation, water) classes that might have passed to the negative axis.

The obtained transformation is visually attractive, and allows easy differentiation of urban/crop/water areas on Sentinel 2 images.

## Script creation process

We have created our model in Python using LDA method of the scikit-learn library [[1]](#ref1). For the input data, we have taken around 80 images of Sentinel-2 L1C and land use classification from Siose. All the images where taken from the Spanish region of La Rioja (with dates ranging from 2015 to 2019). We have tested the estimated model in different world regions (as you can see in our collected [examples](https://github.com/PANOimagen/eobrowser-land-use-visualization/tree/master/examples)) and find it to be visually satisfying, even though the model has been trained only with images from a small geographical region.

The script consist of three components (RGB), each component is created by applying a different LDA to reduce the data to a single dimension for the corresponding color channel.

#### Red channel - Urban
The first component has the data of urban areas. We selected bands usually present when visualizing urban areas (B02, B03, B04, B11 and B12) [[2]](#ref2). The classes we use are `city`, `industry`, `roads` and `non-urban`.

#### Green channel - Vegetation
The second component collects data from vegetation areas. The bands used to calculate vegetation indexes such as NDVI, false infrared color, SAVI, BSI, etc., are B02, B03, B04, B08, B8A and B11 (see in [[2]](#ref2) and [[3]](#ref3)). The classes we use are `crops`, `forests`, `urban green areas`, `arable land` and `non-vegetation`.

#### Blue channel - Water
Finally, the third component has the data of water zones. We use the bands B03, B04, B08 and B11, which are the ones use to calculate indexes such as NDSI (snow), NDWI (water), and NDGI (glaciers) [[3]](#ref3). The classes we used are `rivers`, `lakes` and `seas` and `non-water`.

Before applying LDA we balance the classes, that is, we randomly choose the same number of pixels for each class. Therefore, we have used 4409380, 2128230, 41856 pixels to create each component, respectively. After applying LDA, we center the LDA result (in the range 0-1) and adjust contrast, so that the "non-class" values of each component are zero (using a linear transformation and calculating the absolute value of each component).

## Experimental results

Given the way that the script is designed, it is expected that the urban areas will appear in red, the vegetation in green tones and the areas of water in blue, 
which appears to be the standard behaviour of the script. However we also see the appearance of red in certain crops, and some rivers might also appear in tones other than blue.

In general, the colors for each zone are:

- Urban areas: pink, orange.
- Industrial: brigth purple.
- Crops: purple and bright green tones.
- Forests: dark greens.
- No vegetation: yellow and pink.
- Water: blue.
- Snow: white.

Below we show an image of Madrid on 26-09-2019. More images can be found in the [examples](https://github.com/PANOimagen/eobrowser-land-use-visualization/tree/master/examples) page.

![image](https://github.com/PANOimagen/eobrowser-land-use-visualization/blob/master/examples/Madrid_2019-09-26.jpg?raw=true)


## References

<a name="ref1"></a>[1] Scikit-learn, [Linear Discriminant Analysis (LDA)
 ](https://scikit-learn.org/stable/modules/generated/sklearn.discriminant_analysis.LinearDiscriminantAnalysis.html#sklearn.discriminant_analysis.LinearDiscriminantAnalysis). Accessed on November 2019.
 
<a name="ref2"></a>[2] GitHub repository, [Collection of custom scripts](https://github.com/sentinel-hub/custom-scripts). Accessed on November 2019.
 
<a name="ref3"></a>[3] [List of spectral indexes for Sentinel and Landsat](http://www.gisandbeers.com/listado-indices-espectrales-sentinel-landsat/). Accessed on December 2019.

<a name="ref4"></a>[4] Borràs, J. & Delegido, Jesús & Pezzola, Alejandro & Pereira, M. & Morassi, G. & Camps-Valls, Gustau. (2017). [Clasificación de usos del suelo a partir de imágenes Sentinel-2.](https://www.researchgate.net/publication/317715999_Clasificacion_de_usos_del_suelo_a_partir_de_imagenes_Sentinel-2) Revista de Teledetección. Accessed on November 2019.

<a name="ref5"></a>[5] Pirotti, Francesco & Sunar, Filiz & Piragnolo, M.. (2016). [BENCHMARK OF MACHINE LEARNING METHODS FOR CLASSIFICATION OF A SENTINEL-2 IMAGE.](https://www.researchgate.net/publication/314501156_BENCHMARK_OF_MACHINE_LEARNING_METHODS_FOR_CLASSIFICATION_OF_A_SENTINEL-2_IMAGE) ISPRS - International Archives of the Photogrammetry, Remote Sensing and Spatial Information Sciences. Accessed on December 2019.

<a name="ref6"></a>[6] Anaya Isaza, Andres & Peluffo, Diego & Alvarado Pérez, Juan & Rios, Jorge & Castro Silva, Juan Antonio & Rosero, Paul & Peña, Diego & Salazar Castro, Jose & Umaquinga, Ana. (2016). [Estudio comparativo de métodos espectrales para reducción de la dimensionalidad: LDA versus PCA.](https://www.researchgate.net/publication/311450410_Estudio_comparativo_de_metodos_espectrales_para_reduccion_de_la_dimensionalidad_LDA_versus_PCA_Comparative_study_between_spectral_methods_for_dimension_reduction_LDA_versus_PCA) Accessed on December 2019.


This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">
<img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a>



