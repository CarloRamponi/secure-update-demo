# Demo manifests

* `demo_manifest_1.json`
    - sbom: zlib vulnerable version
    - proof: memset, local verification

* `demo_manifest_2.json`
    - sbom: zlib safe version
    - proofs: memset (correct), local verification

* `demo_manifest_3.json`
    - sbom: zlib safe version
    - proof: memset (wrong), local verification

* `demo_manifest_4.json`
    - sbom: zlib safe version
    - proofs: memset (correct) local, memcpy (correct) remote
