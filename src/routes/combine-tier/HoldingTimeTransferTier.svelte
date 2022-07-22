<script lang="ts" type="module">
  import { signer, signerAddress } from "svelte-ethers-store";
  import Button from "../../components/Button.svelte";
  import FormPanel from "../../components/FormPanel.svelte";
  import Input from "../../components/Input.svelte";
  import { addressValidate } from "../../validation";
  import Switch from "src/components/Switch.svelte";
  import ContractDeploy from "src/components/ContractDeploy.svelte";
  import {
    CombineTier,
    CombineTierGenerator,
    ERC20TransferTier,
  } from "rain-sdk";
  import HumanReadable from "../../components/FriendlySource/HumanReadable.svelte";
  import { queryStore } from "@urql/svelte";
  import { client } from "src/stores";

  let tierContract: string = "0x2554d24a599decd2a1d5395ff9c78dc92d8ed24e",
    multipleCheck = false,
    FriendlySource,
    deployPromise: any,
    tierContractError = null,
    transferTier;

  let blocks: any = [];

  $: transferTier = queryStore({
    client: $client,
    query: `
        query ($tierContract: Bytes!) {
          erc20TransferTier (id: $tierContract) {
            address
          }
        }`,
    variables: { tierContract },
    requestPolicy: "network-only",
  });

  $: tierContractError =
    !$transferTier?.fetching && $transferTier?.data?.erc20TransferTier == null
      ? "Not a valid ERC20TransferTier Contract Address"
      : null;

  $: FriendlySource = {
    tierContract,
    blocks,
  };

  const deployCombineTier = async () => {
    const combineTierConfig = new CombineTierGenerator(
      tierContract
    ).isTierHeldFor(blocks);
    const newCombineTier = await CombineTier.deploy($signer, combineTierConfig);

    return newCombineTier;
  };

  const handleClick = () => {
    deployPromise = deployCombineTier();
  };
</script>

<div class="flex w-full gap-x-3">
  <div class="flex w-3/5 flex-col gap-y-4">
    <div class="mb-2 flex flex-col gap-y-2">
      <span class="text-2xl"
        >Deploy a new "Has Tier Been Held For Blocks" from a TransferTier.</span
      >
      <span class="text-gray-400">
        Choose a TransferTier contract address to deploy a CombineTier that will
        produce a report out of if the tiers have been held for x number of
        blocks or not.
      </span>
    </div>
    <FormPanel heading="Settings">
      <Input
        type="address"
        placeholder="Tier address"
        bind:value={tierContract}
        validator={addressValidate}
        errorMsg={tierContractError}
      >
        <span slot="label">TransferTier Contract</span>
      </Input>
    </FormPanel>
    <FormPanel>
      <div>
        <span> Tier Blocks: </span>
        <Switch
          bind:checked={multipleCheck}
          on:change={() => {
            if (multipleCheck) {
              document.getElementById("multiple").style.display = "block";
              document.getElementById("single").style.display = "none";
            } else {
              document.getElementById("multiple").style.display = "none";
              document.getElementById("single").style.display = "block";
            }
          }}
        />
        <br />
        <span class="text-gray-400"
          >if switched off only one value will be used for all 8 tiers to
          compare against TransferTier report.<br />
          if switched on each tier can have its own value to be compared agaoanst
          TransferTier report, however each provided value should be less than or
          equal to the previous one.</span
        >
      </div>
      <div id="single" style="display:none">
        <Input type="number" placeholder="Tier 1" bind:value={blocks[0]}>
          <span slot="label"
            >Set the time span in blocks for each to cpare against the
            TransferTier report.</span
          >
        </Input>
      </div>
      <div
        id="multiple"
        class="flex w-full flex-col space-y-3"
        style="display:none"
      >
        <Input type="number" placeholder="Tier 1" bind:value={blocks[0]}>
          <span slot="label"
            >Set the time span in blocks for each tier which will be compared
            against the TransferTier report.</span
          >
        </Input>
        <Input type="number" placeholder="Tier 2" bind:value={blocks[1]} />
        <Input type="number" placeholder="Tier 3" bind:value={blocks[2]} />
        <Input type="number" placeholder="Tier 4" bind:value={blocks[3]} />
        <Input type="number" placeholder="Tier 5" bind:value={blocks[4]} />
        <Input type="number" placeholder="Tier 6" bind:value={blocks[5]} />
        <Input type="number" placeholder="Tier 7" bind:value={blocks[6]} />
        <Input type="number" placeholder="Tier 8" bind:value={blocks[7]} />
      </div>
    </FormPanel>

    <FormPanel>
      {#if !deployPromise}
        <Button shrink on:click={handleClick} disabled={tierContractError}
          >Deploy CombineTier</Button
        >
      {:else}
        <ContractDeploy {deployPromise} type="CombineTier" />
      {/if}
    </FormPanel>
  </div>
  <div class="flex w-2/5 flex-col gap-y-4">
    {#if FriendlySource}
      <span class="sticky">
        <FormPanel heading="Human Readable Source">
          <HumanReadable
            signer={$signerAddress}
            contractType="combineTier"
            {FriendlySource}
          />
        </FormPanel>
      </span>
    {/if}
  </div>
</div>

<style>
  span.sticky {
    margin-top: 83px;
    float: right;
    position: sticky;
    top: 90px;
    padding: 5px;
  }
</style>
