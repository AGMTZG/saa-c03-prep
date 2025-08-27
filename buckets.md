# Buckets

## Basic operations with S3 Buckets

### List all buckets

---

<details>
<summary>Show commands / answers</summary>
<p>
  
```bash
aws s3 ls
```

</p>
</details>

### Create a bucket

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
aws s3 mb s3://<your bucket name>
```

</p>
</details>

### Delete a bucket

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
aws s3 rb s3://<your bucket name>
```

</p>
</details>

### Force delete a bucket

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
aws s3 rb s3://<your bucket name> --force
```

</p>
</details>

## Operations with objects

### Create an index.html, css/index.css, js/index.js and upload them to a s3 bucket, you can use the next commands:

```bash
echo "Hello World" > $(pwd)/index.html
sudo mkdir css
sudo mkdir js
cd css
sudo touch index.css
cd ..
cd js
sudo touch index.js
```

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
aws s3 cp index.html s3://<your bucket name>
```

</p>
</details>


### List all objects in your s3 bucket

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
aws s3 ls s3://<your bucket name>
```

</p>
</details>

### List all objects in your s3 bucket using a recursive list(Including folders and subfolders)

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
aws s3 ls s3://<your bucket name> --recursive
```

</p>
</details>

### Upload a folder including subfolders to the s3 bucket

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
aws s3 cp ./myfolder/ s3://<your bucket name>/myfolder/ --recursive
```

</p>
</details>

### Download a file from your s3 bucket

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
aws s3 cp s3://<your bucket name>/index.html ./index.html
```

</p>
</details>

### Download a folder from you s3 bucket

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
aws s3 cp s3://<your bucket name>/folder ./folder --recursive
```

</p>
</details>

### Make changes to the index.html and synchronize your local folder with the files in your s3 bucket

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
aws s3 sync ./ s3://<your bucket name>/
```

</p>
</details>

### Copy the index html in another s3 bucket

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
aws s3 cp s3://<your bucket name source>/<your file> s3://<your bucket name destination>/
```

</p>
</details>

### Move the index.css to another s3 bucket

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
aws s3 mv s3://<your bucket name source>/index.css s3://<your bucket name destination>
```

</p>
</details>

### Erase files in your s3 bucket

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
aws s3 rm s3://<your bucket name>/index.html
or
aws s3 rm s3://<your bucket name> --recursive
```

</p>
</details>
