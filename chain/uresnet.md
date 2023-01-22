## 1. Network architecture
### Basic UResNet
![uresnet_architecture.png](figures/architectures/uresnet_architecture.png)

### Deghosting
![uresnet_deghosting_architecture(1).png](figures/architectures/uresnet_deghosting_architecture(1).png)

## 2. Performance
### PILArNet dataset (w/o ghost points)
From NIPS 2020 workshop paper, by François
![uresnet_noghost.png](figures/performance/pilarnet/uresnet_noghost.png ":size=500")

### ICARUS simulation (w/ ghost points)
As of January 2021, by Laura

![uresnet_icarus_confusion_matrix.png](figures/performance/icarus/uresnet_icarus_confusion_matrix.png ":size=500")

## 3. Michel analysis
Finding Michel electrons using UResNet semantic segmentation output only is done as follows:
1. Extract track-like and Michel-like semantic predictions.
2. Run DBSCAN to form particle clusters.
3. Identify Michel candidates that are touching the end of a muon track cluster

Pixel count spectrum using ICARUS simulation (2020):

![michel_spectrum_after_deghosting.png](figures/performance/michel_spectrum_after_deghosting.png ":size=500")

## 4. Event displays 
(HTML? +CSV? container + config file + run script?)

![data(1).png](figures/event_displays/ghost/data(1).png ":size=200") ![ghost_labels.png](figures/event_displays/ghost/ghost_labels.png ":size=200") ![semantic_labels.png](figures/event_displays/ghost/semantic_labels.png ":size=200")


