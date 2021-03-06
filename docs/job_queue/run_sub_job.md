## Job queue. Run sub job

It shows how you can create and run a sub job, which it is executed separately. 
You can create as many sub jobs as you like. 
They will be executed in parallel. 

```php
<?php
use Enqueue\Client\MessageProducerInterface;
use Enqueue\Psr\Message;
use Enqueue\Psr\Context;
use Enqueue\Psr\Processor;
use Enqueue\JobQueue\JobRunner;
use Enqueue\JobQueue\Job;
use Enqueue\Util\JSON;

class RootJobProcessor implements Processor
{
    /** @var JobRunner */
    private $jobRunner;
    
    /** @var  MessageProducerInterface */
    private $producer;

    public function process(Message $message, Context $context)
    {
        $result = $this->jobRunner->runUnique($message->getMessageId(), 'aJobName', function (JobRunner $runner) {
            $runner->createDelayed('aSubJobName1', function (JobRunner $runner, Job $childJob) {
                $this->producer->send('aJobTopic', [
                    'jobId' => $childJob->getId(),
                    // other data required by sub job
                ]);
            });

            return true;
        });

        return $result ? self::ACK : self::REJECT;
    }
}

class SubJobProcessor implements Processor
{
    /** @var JobRunner */
    private $jobRunner;

    public function process(Message $message, Context $context)
    {
        $data = JSON::decode($message->getBody());

        $result = $this->jobRunner->runDelayed($data['jobId'], function () use ($data) {
            // do your job

            return true;
        });

        return $result ? Result::ACK : Result::REJECT;
    }
}
```

[back to index](../index.md)