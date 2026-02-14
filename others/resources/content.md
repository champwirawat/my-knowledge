# Resources

## SOPS

**age-keygen**

```sh
# Generate age key for SOPS
age-keygen -o sops.agekey

# Generate public key
age-keygen -y sops.agekey > ./secrets/<user_name>.pub

# Encrypt environment variables
age -r $(cat secrets/*.pub) -o .env.enc .env

# Decrypt environment variables (for development)
age -d -i sops.agekey -o .env .env.enc
```

## Programming

- [AI SDK](https://www.youtube.com/watch?v=BQmbuEClULY)
- [NestJS with Telemetry](https://www.tomray.dev/nestjs-open-telemetry)

## Database

- [Drizzle Read Replicas](https://orm.drizzle.team/docs/read-replicas)

## Tools

- [Dev Tool](https://dev-tool.dev/?fbclid=IwY2xjawMI6z9leHRuA2FlbQIxMABicmlkETFWRTdlRHBPTk5LaktLQUg2AR6Nc-MSSM0mezM_ggX88xBax3qBBVPz86OwNDsLrWrD0pJ2DQtg7G-JlrngiA_aem_WYgMk_BjDKaYYIzm3Lmokw)

## Other

- [Uninstall FortiClient on macOS](https://community.fortinet.com/t5/FortiGate/Technical-Tip-How-to-uninstall-FortiClient-on-macOS/ta-p/229617)
