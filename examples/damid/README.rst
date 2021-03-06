The original Kechris et al. 2010 paper finds enrichment regions from
from data generated by the DamID technology followed by tiling arrays.
Here, we attempt to recreate their results.

The original data is available from:
http://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE24024

Their p-values ::

    wget http://amc-sandbox.ucdenver.edu/~brentp/2012/comb-p/data/probes.sorted.bed


We run `comb-p` on the p-values with a seed of 0.1, indicating that we'll
require an slk and FDR corrected p-value of 0.1 or less to start a region.
A maximum distance (--dist) of 1600 indicates that it will look 1600 bases
for another p-value of 0.1 or less to extend the region.

of 1600. ::

    comb-p pipeline \
        --seed 0.1 \
        -p cnew \
        -c 4 \
        --dist 1600 \
        probes.sorted.bed

We then compare the regions from Kechris' paper to compare to the ones founds
with `comb-p`

::

    wget http://amc-sandbox.ucdenver.edu/~brentp/2012/comb-p/data/kechris.regions.be

We count the number of regions per file::

    $ grep -cv "#" cnew.regions-p.bed 
    1248 cnew.regions-p.bed
         
    $ wc -l kechris.regions.bed 
    878 kechris.regions.bed

And the intersections::

    $ intersectBed -b cnew.regions-p.bed -a kechris.regions.bed -u | wc -l
    695

So 695 of Kechris' 878 regions (79%) are represented in the `comb-p` regions.
However, we can see that the regions of overlap have a much higher
enrichment score, again indicating that the regions that overlap are higher
quality::

    $ intersectBed -b cnew.regions-p.bed -a kechris.regions.bed -u \
        | awk '{ sum+=$4 }END{ print sum/NR}'
    2.25478

Than those that do not overlap::

    $ intersectBed -b cnew.regions-p.bed -a kechris.regions.bed -v \
        | awk '{ sum+=$4 }END{ print sum/NR}'
    1.21027

In addition, the ttest on those numbers has a 2-sided p-value of
2.98e-59, indicating that the overlapping set is much different
than the set that does not overlap.

We can also filter the `comb-p` regions by p-value and check the overlap.
83 regions have a corrected p-values less than 0.01::

   $ awk '$7 < 0.01' cnew.regions-p.bed | wc -l
   83

All of those intersect with Kechris regions::

    $ awk '$7 < 0.01' cnew.regions-p.bed \ 
            | intersectBed -a - -b kechris.regions.bed -u | wc -l
    83

This is true all the way up to a p-value of less than 1. ::

    $ awk '$7 < 1' cnew.regions-p.bed | wc -l
    413

    $ awk '$7 < 1' cnew.regions-p.bed \
        | intersectBed -a - -b kechris.regions.bed -u | wc -l
    409

So 409 of 413 of the regions with a corrected p-value less than 1
occur in Kechris' regions. This indicates that the p-value is a good
indicator of confidence.

