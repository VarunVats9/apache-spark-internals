= ReliableRDDCheckpointData

*ReliableRDDCheckpointData* is a rdd:RDDCheckpointData.md[RDDCheckpointData] for ROOT:rdd-checkpointing.md#reliable-checkpointing[Reliable Checkpointing].

== [[creating-instance]] Creating Instance

ReliableRDDCheckpointData takes the following to be created:

* [[rdd]] rdd:RDD.md[++RDD[T]++]

ReliableRDDCheckpointData is created for rdd:RDD.md#checkpoint[RDD.checkpoint] operator.

== [[cpDir]][[checkpointPath]] Checkpoint Directory

ReliableRDDCheckpointData creates a subdirectory of the ROOT:SparkContext.md#checkpointDir[application-wide checkpoint directory] for <<doCheckpoint, checkpointing>> the given <<rdd, RDD>>.

The name of the subdirectory uses the rdd:RDD.md#id[unique identifier] of the <<rdd, RDD>>:

[source,plaintext]
----
rdd-[id]
----

== [[doCheckpoint]] Checkpointing RDD

[source, scala]
----
doCheckpoint(): CheckpointRDD[T]
----

doCheckpoint rdd:ReliableCheckpointRDD.md#writeRDDToCheckpointDirectory[writes] the <<rdd, RDD>> to the <<cpDir, checkpoint directory>> (that creates a new RDD).

With ROOT:configuration-properties.md#spark.cleaner.referenceTracking.cleanCheckpoints[spark.cleaner.referenceTracking.cleanCheckpoints] configuration property enabled, doCheckpoint requests the ROOT:SparkContext.md#cleaner[ContextCleaner] to core:ContextCleaner.md#registerRDDCheckpointDataForCleanup[registerRDDCheckpointDataForCleanup] for the new RDD.

In the end, doCheckpoint prints out the following INFO message to the logs and returns the new RDD.

[source,plaintext]
----
Done checkpointing RDD [id] to [cpDir], new parent is RDD [id]
----

doCheckpoint is part of the rdd:RDDCheckpointData.md#doCheckpoint[RDDCheckpointData] abstraction.
