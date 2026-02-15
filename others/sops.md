# SOPS (age-keygen)

SOPS ใช้สำหรับเข้ารหัสไฟล์ env และ secrets ด้วย age keys

--------------------------------------------------------------------------------

## ⚙️ การใช้งาน

### 1\. สร้าง age key

```sh
# Generate age key for SOPS
age-keygen -o sops.agekey

# Generate public key
age-keygen -y sops.agekey > ./secrets/<user_name>.pub
```

### 2\. เข้ารหัสและถอดรหัส environment variables

```sh
# Encrypt environment variables
age -r $(cat secrets/*.pub) -o .env.enc .env

# Decrypt environment variables (for development)
age -d -i sops.agekey -o .env .env.enc
```

--------------------------------------------------------------------------------

## 📚 Learning Resources

- [SOPS - Mozilla](https://github.com/getsops/sops)
- [age - Encryption Tool](https://github.com/FiloSottile/age)
