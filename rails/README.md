
# Rails ONBUILD Dockerfile

## Building

docker build -t phatforge/rails:onbuild .

## Tagging

Speculative

Tag these so they can be pushed to private docker registries

docker tag phatforge/rails:onbuild phatforge/rails:2.0.0-p598
docker tag phatforge/rails:onbuild images.phatforge.com/phatforge/rails
docker tag phatforge/rails:onbuild images.phatforge.com/phatforge/rails:onbuild
