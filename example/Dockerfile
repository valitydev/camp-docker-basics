# Compilaton and testing

FROM erlang:21.0 as build

COPY ./ /build
WORKDIR /build

RUN rebar3 compile
RUN rebar3 do eunit, ct

RUN rm -rf _build/default/rel
RUN rebar3 release

# Make an app image

FROM erlang:21.0

WORKDIR /opt/dummy_docker
COPY --from=build /build/_build/default/rel/dummy_docker .

CMD bin/dummy_docker foreground
