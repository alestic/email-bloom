# email-bloom

## NAME

email-bloom - Use a Bloom filter to compare email lists anonymously
between two parties


## SYNOPSIS

`email-bloom [options] [FILE]...`

## OPTIONS

`-h|--help` - Show help (progam usage) and exit

`-v|--version` - Show program version and exit

One and only one of `--output-bloom` or `--input-bloom` must be
specified:

`--output-bloom BLOOMFILE` - Generate Bloom filter from input files
and write to BLOOMFILE

`--input-bloom BLOOMFILE` - Use Bloom filter in BLOOMFILE and output
any matching email addresses from input files

`--max-false-positive-rate RATE` - The desired max false positive rate
when generating an output Bloom filter file, between 0 and 1. Not used with
`--input-bloom`. (Default: 0.01 = 1%)

## ARGUMENTS

`FILE` - Zero or more files that contain one email address per
line. If no files are specified, stdin is read.

## DESCRIPTION

### Scenario

Party A and Party B each have a list of email addresses, but neither
wants to share their entire list with the other party, especially not
the email addresses that the other party does not have.

Both parties want Party B to identify which of Party A's email
addresses are in Party B's list (e.g., unsubscribe). They are willing
to have some small percentage of additional email addresses in Party
B's list identified as matches even though they were not actually in
Party A's email address list (false positive rate).

### Approach

The anonymous cross-party matching is done using a Bloom filter, which
can identify items that are probably in a set and items that are
definitely not in a set.

For more information on Bloom filters, please read:

> <https://en.wikipedia.org/wiki/Bloom_filter>

Party A creates a Bloom filter using the email addresses in their
list, specifying the false positive rate they would like Party B to
experience when comparing items.

Party A passes the Bloom filter to Party B, who applies it to each
email address in their list, identifying which of Party B's email
addresses are probably in Party A's list, and which are definitely
not.

## EXAMPLES

1. Party A generates a Bloom filter using their email list:

        email-bloom \
          --max-false-positive-rate 0.01 \
          --output-bloom emails-a.bloom \
          emails-a.txt

   The input file (emails-a.txt) has one email address per line. The
   max false positive rate indicates what fraction of Party B's email
   addresses they want to be identified as matching, even thought they
   are not. A false positive rate of 0.01 indicates that up to 1% of
   email addreses will be identified as matching even though they are
   not.

2. Party A passes the Bloom filter file (emails-a.bloom) to Party B.

3. Party B runs their email address list (emails-b.txt) through the
   Bloom filter from Party A (emails-a.bloom):

        email-bloom \
          --input-bloom emails-a.bloom \
          emails-b.txt \
          > emails-probably-in-a.txt

   The output definitely includes all email addresses that are in
   common, plus some percentage that are not (false positive
   rate). Any email addresses in Party B's emails-b.txt that are not
   output from this command are definitely not in Party A's
   emails-a.txt

## REQUIREMENTS

This program uses the `pybloom` Python package, which may be installed
with:

    pip install pybloom

## SEE ALSO

Bloom Filters: <https://en.wikipedia.org/wiki/Bloom_filter>

## CAVEATS

This is pre-beta software, not peer-reviewed, and not extensively
tested. Not recommended for high value, high risk applications. Use at
your own risk.

Though the goal of this program (and of Bloom filters) is to hide some
information while providing the ability to determine other
information, it may contain defects and uses third party software
(pybloom) that may also contain defects. It is your responsibility to
make sure that a Bloom filter is appropriate for your intended usage,
and to verify that this program is working how you want it to work for
your use case.

The lower the false positive rate, the more likely it is that Party B
can find email addresses likely to be in Party A's list, even if they
do not have the same email address.

Even with a high false positive rate, it may be possible for Party B
to determine general information about Party A's list based on what
types of email addresses tend to be marked as matches.

There can be multiple ways to specify the same email address that are
not caught by this program. For example, example@gmail.com,
e.xample@gmail.com, and example+fun@gmail.com are all the same
address, but will not (currently) be matched.

Input lines have leading and trailing whitespace trimmed, and are
lowercased to help match email addresses. Other email address
canonicalization operations may be performed, possibly making this
tool inappropriate for usage with matching data other than email
addresses.

## MAINTAINER

Eric Hammond `<ehammond@thinksome.com>`

## LICENSE

Copyright 2017 Eric Hammond `<ehammond@thinksome.com>`

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

 http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
