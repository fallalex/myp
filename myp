#!/usr/bin/env python3

"""This is a  common question when working on a computer "What is the public IP?"
There are answers out there but most are aimed towards a users Everyday PC
As I got into VMs from different cloud providers and set up a server rack at home
knowing public IPs became more important for setting DNS records and firewall whitelists
I wanted a service that was more resiliant than a curl command to a single server.
Ideally you would want to self host a service to get public IPs from different locations
but if you do need to rely on external hosts you should have a few backups

This program has a bunch of fallbacks to decease the chance of coming up empty handed.
it uses:
    o-o.myaddr.l.google.com (resolved using a google nameserver like ns1.google.com)
    myip.opendns.com (resolved using a opendn nameserver like resolver1.opendns.com)
    checkip.amazonaws.com
    icanhazip.com

It is based off of commands like the following:
    dig +short A myip.opendns.com @resolver1.opendns.com
    dig +short TXT o-o.myaddr.l.google.com @ns1.google.com
    dig whoami.akamai.net. @ns1-1.akamaitech.net. +short
    nslookup -type=txt o-o.myaddr.l.google.com ns1.google.com
    curl checkip.amazonaws.com

The goal is not to check that the public IP returned by different services is the same.
In order to keep the run time low this goes through the different options sequentially
when an IP Address is recieved the programs job is done and it will exit.
"""

import dns.resolver
import ipaddress
import sys
import requests

google = ('8.8.8.8','8.8.4.4')
opendns = ('208.67.222.222','208.67.222.220')

# the google nameservers below are not the same as thier public DNS servers
# these must be used to access o-o.myaddr.l.google.com
nameservers_google = ('ns1.google.com','ns2.google.com')
nameservers_opendns = ('resolver1.opendns.com','resolver2.opendns.com')

def public_ip_dns(resolv, nameservers, rdatatype, server, responsetype):
    """Use a basic Resolver to find the IP Addresses of specific nameservers
    This allows the porgram to be less dependant on hard coded IP Addresses"""
    for ns in nameservers:
        try:
            answer = resolv.query(ns, rdatatype)
            nameserver = answer[0].to_text()
        except Exception as e:
            print(e)
            continue
        resolve_public_ip(nameserver, server, responsetype)

def resolve_public_ip(nameserver, server, responsetype):
    """Using the resolved IP Address of a nameserver, query a specific server
    for the public IP Address of the host running this program"""
    request_resolver = dns.resolver.Resolver()
    request_resolver.nameservers = [nameserver,]
    try:
        answer = request_resolver.query(server, responsetype)
        ip = answer[0].to_text().replace('"','').strip()
        ipaddress.ip_address(ip)
        print(ip)
        sys.exit()
    except Exception as e:
        print(e)
    return None

# Thinking of specifying the DNS to use
def public_ip_url(url, sanitize=(lambda x:x)):
    """Send HTTP GET to a URL, parse and verify the reply"""
    reply = requests.get(url)
    if reply.status_code == 200:
        try:
            ip = sanitize(reply.text.strip())
            ipaddress.ip_address(ip)
            print(ip)
            sys.exit()
        except Exception as e:
            print(e)

def main():
    # on my network IPv6 will not resolve 'ping6 google.com'
    # so right now this only works for IPv4 but IPv6 will work

    # GET from HTTP servers
    public_ip_url('http://icanhazip.com')
    public_ip_url('http://checkip.amazonaws.com')

    # Simple DNS check
    resolve_public_ip(opendns[0], 'myip.opendns.com', 'A')

    # More robust DNS options
    ns_resolver = dns.resolver.Resolver()
    ns_resolver.nameservers = opendns + google
    public_ip_dns(ns_resolver, nameservers_opendns, 'A', 'myip.opendns.com', 'A')
    public_ip_dns(ns_resolver, nameservers_google, 'A', 'o-o.myaddr.l.google.com', 'TXT')


if __name__ == "__main__":
    main()
