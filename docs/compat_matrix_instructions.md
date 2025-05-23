# PQC Interoperability Matrix Generation

This tool parses interoperability results files and creates Markdown-based
tables.

The `rebuild_results.sh` script will compile the interop test results from all providers.
If you have updated your test results, then please re-run `rebuild_reselts.sh` before committing.
https://ietf-hackathon.github.io/pqc-certificates/pqc_hackathon_results.html displays the /docs folder of the `master` branch.

## The interoperability results format R5

Results for the R5 *certs* tests should be placed in
`providers/<provider_name>/compatMatrices/artifacts_certs_r5/<producer>_<consumer>.csv`.
Because there in this format there can be multiple files for each OID (up to
three private key formats, a certificate, and for ML-KEM possible ciphertext
and shared secret), there are now three columns in the CSV file.  For example,
the results for ML-DSA-44 and SLH-DSA-SHA2-128s might be:

```
key_algorithm_oid,type,test_result
2.16.840.1.101.3.4.3.17,cert,Y
2.16.840.1.101.3.4.3.17,seed,Y
2.16.840.1.101.3.4.3.17,expandedkey,Y
2.16.840.1.101.3.4.3.17,both,Y
2.16.840.1.101.3.4.3.17,consistent,Y
2.16.840.1.101.3.4.3.20,priv,Y
2.16.840.1.101.3.4.3.20,cert,Y
```

The `type` column takes the following values:

- cert
    - The TA certificate (ML-DSA or SLH-DSA) had a valid self-signature, or
    - The ML-KEM EE certificate had a valid signature, by the associated ML-DSA TA.
- priv
    - The single SLH-DSA private key form was well formed.
    - A signature made with the private key can be verified with the
      corresponding public key.
- seed
    - The seed form of an ML-DSA or ML-KEM private key form was well formed.
    - A signature made with that ML-DSA private key can be verified with the
      corresponding public key.
    - The encapsulation ciphertext provided for that ML-KEM OID decapsulates
      via the seed-only key to a shared secret identical with the provided
      shared secret.
- expandedkey
    - The expanded key form of an ML-DSA or ML-KEM private key form was well
      formed.
    - A signature made with that ML-DSA private key can be verified with the
      corresponding public key.
    - The encapsulation ciphertext provided for that ML-KEM OID decapsulates
      via the expanded key to a shared secret identical with the provided
      shared secret.
- both
    - The seed plus expanded key (both) form of an ML-DSA or ML-KEM private key
      form was well formed.
    - A signature made with that ML-DSA private key can be verified with the
      corresponding public key.
    - The encapsulation ciphertext provided for that ML-KEM OID decapsulates
      via the seed plus expanded key form to a shared secret identical with the
      provided shared secret.
- consistent
    - All the private key forms yield the same public key form, that is also
      the public key in the certificate.
    - This test is not applicable if only a certificate and no keys, or only
      one key and no certificate is provided.

## The interoperability results format R3

This is simplified relative to the R2 format.

Results should be placed in `providers/<provider_name>/compatMatrices/<zip_name>/<producer>_<consumer>.csv`.

The header line for the result file has the following columns:

`key_algorithm_oid`, `test_result`

The `key_algorithm_oid` contains the OID contained in the SPKI for the given algorithm.
The `test_result` column contains whether verification of the the specific artifact was successful, failed, or wasn't performed. Successful verifications are denoted with `Y`. Failures are denoted with `N`, and verifications that weren't performed are denoted by the empty string(``).

For example, a result file `X_Y.csv` for implementation Y verifying X's Dilithium2 artifacts will have the following content:

```
key_algorithm_oid,test_result
1.3.6.1.4.1.2.267.7.4.4,Y
```

## The interoperability results format R2

Interoperability result files are CSV-formatted. JSON support is planned.
The result files should be placed in your provider's dir in a compatMatrices/ subdir. The file names have the following format: `PRODUCER_CONSUMER.csv`, where `PRODUCER` is the name of the implementation that generated the artifacts, and `CONSUMER` is the name of the implementation that verified the artifacts.

The header line for the result file has the following columns:

`key_algorithm_oid`, `ta`, `ca`, `ee`, `crl_ta`, `crl_ca`

The `key_algorithm_oid` contains the OID contained in the SPKI for the given algorithm.

The other columns contain whether verification of the the specific artifact was successful, failed, or wasn't performed. Successful verifications are denoted with `Y`. Failures are denoted with `N`, and verifications that weren't performed are denoted by the empty string(``).

For example, a result file `X_Y.csv` for implementation Y verifying X's Dilithium2 artifacts will have the following content:

```
key_algorithm_oid,ta,ca,ee,crl_ta,crl_ca
1.3.6.1.4.1.2.267.7.4.4,Y,Y,,,
```

In this example, the root and ICA signatures validated, but the end-entity certificate, root CRL, and intermediate CRL were not evaluated.

## Setup

1. Install Python 3
2. Install [mdutils](https://github.com/didix21/mdutils): `pip3 install -r requirements.txt`
3. `apt install pandoc` (for `rebuild_results.sh`).

## Execution

Run `python3 pqc_report_writer.py OID_MAPPING_FILE FILES`, where:

* `OID_MAPPING_FILE` is the file that contains the mapping of OIDs to their long names
* `FILES` is the list of result files

The resulting Markdown file will be output to `pqc_hackathon_results.md`.

The script `rebuild_results.sh` includes calling pqc_report_writer.py, so you can just use that.
