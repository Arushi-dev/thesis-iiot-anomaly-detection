# Data

The raw data is not included in this repository. It comes from the **TON_IoT datasets** (UNSW Canberra), which are free to use for academic research. Download them from the [TON_IoT project page](https://research.unsw.edu.au/projects/toniot-datasets).

The corrected notebooks expect two Zeek `conn` logs, in CSV format with no header row, at:

```
data/normal_2/conn.csv          # normal traffic, 2 April 2019 (7,179 connections)
data/normal_DDoS_1/conn.csv     # DDoS attack window with background traffic, 25 April 2019 (123,536 connections)
```

The original notebooks in `notebooks/original/` used absolute Windows paths and intermediate files (`df_normal_processed.csv`, `df_ddos_processed.csv`) that are too large for GitHub.
