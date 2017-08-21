# Title: buildkite kolibri builds failing due to upgraded docker image

# Events:
  - When built on the docker 
  - Mr. A recommendated replacing the docker image 

# cause

# short term fix

# long term solution
 - docker import export
 - don't do docker pull; 
 - use image digests (recommended) or use a specific tag with a date (`ubuntu:xenial-20170802`)
