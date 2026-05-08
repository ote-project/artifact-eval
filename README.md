# Artifact Evaluation README

**Paper:** Extracting Database Access-control Policies From Web Applications (OSDI '26)

This repository contains the artifact for evaluating Ote, a system that extracts database access-control policies for Ruby-on-Rails web applications. Ote explores application execution paths, records database queries and path conditions, and produces access-control policies expressed as SQL views.

The artifact supports two run modes:

- `once`: abridged run. Runs each experiment once. This takes approximately **8 hours**.
- `full`: full run as reported in the paper. Runs each experiment three times and takes the median run.

Ote uses an LLM-based relevance judge implemented using OpenAI's [Codex CLI](https://developers.openai.com/codex/cli). To conserve resources, the artifact by default uses a mock relevance judge with an artificially imposed latency of 90 seconds to simulate real performance.

## Hardware requirements

Run the artifact on [Google Cloud](https://cloud.google.com/) using:

- [Compute Engine](https://cloud.google.com/products/compute) machine type: `c3-standard-176`

## Download and reassemble the VM image

The VM image is distributed as split `xz` parts in the GitHub release [`ae`](https://github.com/ote-project/artifact-eval/releases/tag/ae).

To download the image parts using [GitHub CLI](https://cli.github.com/) and reassemble them:

```bash
mkdir -p artifact-image && cd artifact-image && \
gh release download ae -R ote-project/artifact-eval -p '*xz.part*' && \
cat *xz.part* > artifact.vmdk.xz && \
xz -d artifact.vmdk.xz
```

Alternatively, using `curl` and `jq`:

```bash
mkdir -p artifact-image && cd artifact-image && \
wget -i <(curl -s https://api.github.com/repos/ote-project/artifact-eval/releases/tags/ae \
  | jq -r '.assets[] | select(.name | test("xz\\.part")) | .browser_download_url') && \
cat *xz.part* > artifact.vmdk.xz && \
xz -d artifact.vmdk.xz
```

This produces `artifact.vmdk`.

## Import the VMDK into GCP

To create a VM using this image:

1. [Upload](https://docs.cloud.google.com/storage/docs/uploading-objects) the VMDK to a Google Cloud Storage bucket.
2. [Import](https://docs.cloud.google.com/migrate/virtual-machines/docs/5.0/migrate/machine-image-import) it as a machine image.
3. After the import completes, [create a VM instance](https://docs.cloud.google.com/compute/docs/instances/create-start-instance) from the image using machine type `c3-standard-176`.

## SSH into the VM

SSH into the VM using, for example, GCP's [SSH-in-browser feature](https://docs.cloud.google.com/compute/docs/connect/standard-ssh#console) or Google Cloud CLI's [`gcloud compute ssh` command](https://docs.cloud.google.com/sdk/gcloud/reference/compute/ssh).

Then, switch to the `ubuntu` user:

```bash
sudo -iu ubuntu
```

## Run the artifact

Start `tmux`, since the experiment is long-running:

```bash
tmux
```

Then run:

```bash
cd dse/scripts
./run-ae.sh once
```

The abridged `once` mode runs each experiment once and takes approximately 8 hours in total.

For the full run as reported in the paper:

```bash
cd dse/scripts
./run-ae.sh full
```

The `full` mode runs each experiment three times reports the median run.

## Using the real Codex-based relevance judge

By default, the artifact uses a mock relevance judge with an artificially imposed latency of 90 seconds to simulate real performance while conserving resources.

To use the actual Codex-based relevance judge:

1. Log into Codex on the VM.
2. Run the artifact with `--use-codex`.

For example:

```bash
cd dse/scripts
./run-ae.sh once --use-codex
```

## Inspecting the results

When the run completes, the script prints the path to a generated PDF containing the artifact-evaluation tables. It will look like:

```
Wrote: /home/ubuntu/dse/logs/ae-tables-20260507-092535/ae.pdf
```

Download that PDF from the VM and inspect the generated evaluation tables.

## Source code

The source code for the artifact components is available under the [Ote Project GitHub organization](https://github.com/ote-project).

The main components are:

- [`artifact-eval`](https://github.com/ote-project/artifact-eval): artifact-evaluation README and VM image.
- [`concolic_driver`](https://github.com/ote-project/concolic_driver): Ote's concolic-execution driver and policy-generation logic.
- [`jruby`](https://github.com/ote-project/jruby): modified JRuby interpreter used for symbolic tracking.
- [`blockaid`](https://github.com/ote-project/blockaid): policy-enforcement and view-pruning support.
- [`ae-scripts`](https://github.com/ote-project/ae-scripts): scripts for artifact evaluation.
- [`ae-app-config`](https://github.com/ote-project/ae-app-config): application configuration and runners.
- [`Autolab`](https://github.com/ote-project/Autolab): modified Autolab application used in the evaluation.
- [`diaspora`](https://github.com/ote-project/diaspora): modified diaspora application used in the evaluation.
- [`theodinproject`](https://github.com/ote-project/theodinproject): modified The Odin Project application used in the evaluation.
