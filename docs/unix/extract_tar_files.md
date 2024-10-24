# Extracting Tar.gz Files

`.tar.gz` files are a more space-efficient alternatives to compress files than `.zip` files. Github gives the option when a github release is created to download either a `zip` file or a `.tar.gz` file. For space-efficency I typically opt to download a `.tar.gz` file.

## To Extract files from a `.tar.gz` file to current the working directory

```bash
tar -xvf filename.tar.gz
```

For other use cases check out this [article](https://kinsta.com/knowledgebase/unzip-tar-gz/).

You can also run `man tar`
