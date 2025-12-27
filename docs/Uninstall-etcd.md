Got it 👍
Since you installed **etcd manually (Kubernetes the Hard Way style)**, we must remove **everything by hand**: service, binaries, data, certs, and cluster membership.

⚠️ **Run these steps on BOTH nodes**: `controlplane01` and `controlplane02`.

---

# ✅ Complete etcd Uninstall (Clean Reset)

## 1️⃣ Stop and disable etcd service

```bash
sudo systemctl stop etcd
sudo systemctl disable etcd
```

Verify:

```bash
systemctl status etcd
```

---

## 2️⃣ Remove the systemd unit

```bash
sudo rm -f /etc/systemd/system/etcd.service
sudo systemctl daemon-reload
sudo systemctl reset-failed
```

---

## 3️⃣ Delete etcd data directory (CRITICAL)

This removes **all cluster state**.

```bash
sudo rm -rf /var/lib/etcd
```

---

## 4️⃣ Remove etcd binaries

```bash
sudo rm -f /usr/local/bin/etcd
sudo rm -f /usr/local/bin/etcdctl
```

Verify:

```bash
which etcd
which etcdctl
```

(no output = good)

---

## 5️⃣ Remove etcd certificates & symlink

```bash
sudo rm -rf /etc/etcd
```

⚠️ **Important**
Do **NOT** delete `/var/lib/kubernetes/pki/ca.crt` unless you want to regenerate **all Kubernetes certs**.

If you want to remove only the etcd CA symlink:

```bash
sudo rm -f /etc/etcd/ca.crt
```

---

## 6️⃣ Remove leftover etcd files (sanity check)

```bash
sudo find / -name "*etcd*" 2>/dev/null
```

Only Kubernetes-related references should remain later.

---

## 7️⃣ Verify ports are free

```bash
sudo ss -lntp | grep -E '2379|2380'
```

No output = etcd fully removed ✅

---

# 🧼 Optional: Reset system completely (KTH redo)

If you want to **redo Kubernetes the Hard Way from scratch**, also remove:

```bash
sudo rm -rf /var/lib/kubernetes
```

⚠️ This wipes **ALL control-plane state**.

---

# ✅ Ready to reinstall etcd

After cleanup, you can safely re-run:

```bash
07-bootstrapping-etcd.md
```

with:

* Fresh cluster state
* No stale certs
* No membership conflicts

---

## 🧠 Why this matters (KTH gotcha)

* etcd **remembers cluster membership** in `/var/lib/etcd`
* Re-running with `--initial-cluster-state new` **without cleanup breaks quorum**
* Most KTH issues come from **not deleting `/var/lib/etcd`**

---

If you want, next I can:

* Validate your **PRIMARY_IP / DNS / /etc/hosts**
* Help you **re-bootstrap etcd correctly**
* Check **etcd logs if it fails again**

Just say the word 👌

Post that check if etcd is helathy using below command

```
sudo ETCDCTL_API=3 etcdctl endpoint health \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/etcd-server.crt \
  --key=/etc/etcd/etcd-server.key
```
