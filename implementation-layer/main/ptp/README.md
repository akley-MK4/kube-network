# Macvlan
This project is about the use and configuration of the ptp.

## Reference
1. https://www.cni.dev/plugins/current/main/ptp/

## Limitations on use
1. Avoid using PTP for Pod to Pod communication, PTP is designed for point-to-point connections and is not suitable for multi Pod shared networks.
2. Each vet pair is completely independent and does not share a broadcast domain.

