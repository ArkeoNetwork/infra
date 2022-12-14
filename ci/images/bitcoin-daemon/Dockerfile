FROM debian:11-slim AS installer

ENV BITCOIN_VERSION=23.0
# refer to https://github.com/bitcoin/bitcoin/tree/master/contrib/builder-keys for the PGP keys of builders and developers
ENV KEYS 152812300785C96444D3334D17565732E08E5E41 0AD83877C1F0CD1EE9BD660AD7CC770B81FD22A8 590B7292695AFFA5B672CBB2E13FC145CD3F4304 28F5900B1BB5D1A4B6B6D1A9ED357015286A333D 637DB1E23370F84AFF88CCE03152347D07DA627C CFB16E21C950F67FA95E558F2EEB9F5CC09526C1 F4FC70F07310028424EFC20A8E4256593F177720 D1DBF2C4B96F2DEBF4C16654410108112E7EA81F 287AE4CA1187C68C08B49CB2D11BD4F33F1DB499 F9A8737BF4FF5C89C903DF31DD78544CF91B1514 9DEAE0DC7063249FB05474681E4AED62986CD25D E463A93F5F3117EEDE6C7316BD02942421F4889F 9D3CC86A72F8494342EA5FD10A41BDC3F4FAFF1C 4DAF18FE948E7A965B30F9457E296D555E7F63A7 28E72909F1717FE9607754F8A7BEB2621678D37D 74E2DEF5D77260B98BC19438099BAD163C70FBFA
ENV KEYS2 637DB1E23370F84AFF88CCE03152347D07DA627C 82921A4B88FD454B7EB8CE3C796C4109063D4EAF

WORKDIR /work

RUN apt-get update \
    && apt-get -y install --no-install-recommends wget ca-certificates gnupg

RUN wget https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/bitcoin-${BITCOIN_VERSION}-x86_64-linux-gnu.tar.gz \
    && wget https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/SHA256SUMS \
    && wget https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/SHA256SUMS.asc

RUN gpg --keyserver keyserver.ubuntu.com --recv-keys ${KEYS} \
    && gpg --keyserver keys.openpgp.org --recv-keys ${KEYS2} \
    # removing one problematic sig from SHA256SUMS.asc (using sed) for which I could not find a usable public key from any keyservers
    # C388F6961FB972A95678E327F62711DBDCA8AE56
    # kvaciral@protonmail.com
    && sed -i -e '156,162d' SHA256SUMS.asc \
    && gpg --verify SHA256SUMS.asc SHA256SUMS \
    && grep bitcoin-${BITCOIN_VERSION}-x86_64-linux-gnu.tar.gz SHA256SUMS | sha256sum -c \
    && mkdir bitcoin \
    && tar xzvf bitcoin-${BITCOIN_VERSION}-x86_64-linux-gnu.tar.gz --strip-components=1 -C bitcoin

FROM debian:11-slim

COPY --from=installer /work/bitcoin/bin/* /usr/local/bin/
COPY --from=installer /work/bitcoin/lib/* /usr/local/lib/
COPY --from=installer /work/bitcoin/share/* /usr/local/share/

COPY ./scripts /scripts

RUN useradd --create-home bitcoin
USER bitcoin

EXPOSE 8332 8333 18332 18333 18443 18444
VOLUME ["/home/bitcoin/.bitcoin"]

ENTRYPOINT ["/scripts/entrypoint.sh"]
CMD ["bitcoind"]
