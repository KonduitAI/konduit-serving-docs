# Pip package

To install the `konduit` pip package, you can run:

```bash
pip install konduit
```

After installing the pip package, you can initialize the `konduit` CLI in the following way

### For git users

If you use git you can build the binaries by running

#### CPU

```bash
konduit-init --chip cpu
```

#### GPU

```bash
konduit-init --chip gpu
```

### For non-git users

If you want to download the pre-build binaries, you can

#### CPU

```bash
konduit-init --chip cpu -d
```

#### GPU

```text
konduit-init --chip gpu -d
```

## Verification

After doing the above process, you can verify the installation by doing:

```bash
konduit --version
```

If everything went alright then you should see something like

```text
Konduit serving version: 0.1.0-SNAPSHOT
Commit hash: 3f9ac52f
```

## What's next? 

Try out how to work with the `konduit` CLI with an example workflow [here](quickstart-cli.md)

