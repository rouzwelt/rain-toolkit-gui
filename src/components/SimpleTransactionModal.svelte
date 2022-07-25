<script lang="ts">
  import { getContext, onMount } from "svelte";
  import Ring from "./Ring.svelte";
  import { selectedNetwork } from "src/stores";
  import { ContractReceipt } from "ethers";
  import Button from "./Button.svelte";
  import { Logger } from "ethers/lib/utils";

  enum TxStatus {
    None,
    AwaitingSignature,
    AwaitingConfirmation,
    Error,
    Confirmed,
  }

  const { close } = getContext("simple-modal");

  export let method: Function;
  export let args: any[];
  export let confirmationMsg: string;
  export let func: string | undefined = undefined;

  export let returnValue: undefined | any = (method) => {};
  let errorMsg: string, receipt: ContractReceipt, txStatus: TxStatus;

  const closed = () => {
    close();
    if (func == "approve") {
      returnValue(true, receipt);
    } else returnValue(false, receipt);
  };

  onMount(async () => {
    let tx;

    txStatus = TxStatus.AwaitingSignature;

    try {
      tx = await method(...args);

      txStatus = TxStatus.AwaitingConfirmation;
      receipt = await tx.wait();
    } catch (error) {
      if (error.code === Logger.errors.TRANSACTION_REPLACED) {
        if (error.cancelled) {
          errorMsg = "Transaction Cancelled";
          txStatus = TxStatus.Error;
          return;
        } else {
          receipt = await error.replacement.wait();
        }
      } else {
        errorMsg = error.data?.message || error?.message;
        txStatus = TxStatus.Error;
        return;
      }
    }

    txStatus = TxStatus.Confirmed;
  });
</script>

{#if txStatus == TxStatus.AwaitingSignature}
  <div class="flex flex-col items-center gap-y-5 p-6">
    <Ring color="#fff" />
    <span class="text-lg">Awaiting signature...</span>
  </div>
{/if}
{#if txStatus == TxStatus.AwaitingConfirmation}
  <div class="flex flex-col items-center gap-y-5 p-6">
    <Ring color="#fff" />
    <span class="text-lg">Transaction confirming...</span>
  </div>
{/if}
{#if txStatus == TxStatus.Confirmed}
  <div class="flex max-w-md flex-col items-center gap-y-5 p-6">
    <span class="text-lg">Transaction confirmed</span>
    <span>
      {confirmationMsg}
    </span>
    <a
      class="text-blue-400 underline"
      target="_blank"
      href={`${$selectedNetwork.blockExplorer}/tx/${receipt?.transactionHash}`}
    >
      See transaction.
    </a>
    <Button on:click={closed}>Ok</Button>
  </div>
{/if}
{#if txStatus == TxStatus.Error}
  <div class="flex flex-col items-center gap-y-5 p-6">
    <span class="text-lg">Something went wrong.</span>
    <span class="text-lg text-red-400">{errorMsg}</span>
  </div>
{/if}
